# Resource and Infrastructure Troubleshooting

This file documents infrastructure issues encountered during the lab build and how
they were resolved. These are genuine findings from running a multi-VM SOC lab on
a single laptop with limited resources.

## Issue 1 — Host disk full (recurring)

**Symptom**: VirtualBox paused VMs mid-operation with error:
`Host system reported disk full. VM execution is suspended.`

**Root cause**: Host laptop C: drive repeatedly hit 0.00GB free while running
4 VMs simultaneously. Each VM has a dynamically-allocated VDI disk that grows
over time. Combined with downloaded ISOs, Elastic Agent zips from multiple
reinstall attempts, and old VirtualBox snapshot files, available space was
consumed faster than expected.

**Resolution**:
- Deleted Windows Server evaluation ISO after VM was built (4.7GB recovered)
- Deleted old Elastic Agent zip files from multiple download attempts
- Ran Windows Disk Cleanup targeting system files
- Ran `powercfg -h off` to delete hiberfil.sys
- Deleted old Elasticsearch indices from July (6 backing indices, ~300MB)
- Set VirtualBox default machine folder to external drive (when available)

**Key lesson**: On a resource-constrained host, snapshot files are the largest
hidden consumer. Each VirtualBox snapshot can be as large as the VM's used disk
space. Maintain only one recent "known good" snapshot per VM and delete older
ones regularly.

## Issue 2 — Elastic Agent "previously unenrolled" crash loop

**Symptom**: Elastic Agent service starts, then immediately stops. Agent log shows:
`Elastic Agent was previously unenrolled. To reactivate please reconfigure or enroll again.`

**Root cause**: VirtualBox snapshot restore reverts the VM to a point in time
before the current Fleet enrollment. Fleet Server (on `theanalyst`) keeps moving
forward, so after a restore the agent's local state is "old" from Fleet's
perspective and Fleet marks it as unenrolled.

**Resolution**: Re-enroll with a fresh token every time after a snapshot restore:
```powershell
cd "C:\Users\Administrator\elastic-agent-8.11.2-windows-x86_64"
.\elastic-agent.exe uninstall
.\elastic-agent.exe install --url=https://192.168.10.10:8220 `
  --enrollment-token=<FRESH-TOKEN> --insecure
```

**Key lesson**: This will happen every single time a snapshot is restored.
It is expected behavior, not a bug. Always re-enroll after restoring a snapshot.
Take new snapshots immediately after confirming a healthy enrollment state.

## Issue 3 — Winlogbeat subprocess not spawning (Elastic Agent 8.11.x)

**Symptom**: WIN-Sysmon Custom Windows Event Log integration configured correctly
in Fleet policy, agent shows HEALTHY, but no Sysmon events appear in Kibana.
Process list shows filebeat and metricbeat subprocesses but no winlogbeat.

**Root cause**: Elastic Agent 8.11.x has a known compatibility issue where the
winlogbeat subprocess (required for Custom Windows Event Log integrations) does
not spawn correctly in some configurations, particularly after multiple
re-enrollments or policy changes.

**Confirmed by**:
- `Get-Process | Where-Object {$_.Name -like "*beat*"}` — no winlogbeat in list
- `elastic-agent.yml` — no winlog/channel/sysmon entries present
- Sysmon events confirmed locally via `Get-WinEvent` — data exists on the endpoint
- Elasticsearch index `logs-winlog.winlog-default` — only 1 document ever shipped

**Resolution attempted**: Multiple policy re-creations, fresh enrollments, policy
revisions — winlogbeat subprocess still did not spawn.

**Workaround**: Sysmon events confirmed locally via PowerShell. C2 callback and
payload execution confirmed directly via Mythic web UI. Dashboard built based on
known field structure from challenge curriculum. Documented as known 8.11.x issue.

## Issue 4 — Docker iptables REJECT blocking inbound connections (Oracle Cloud)

**Symptom**: Port confirmed listening (`ss -tlnp`), Oracle Security List rule
confirmed present, but all external connection attempts fail. tcpdump showed
SYN packets arriving but no SYN-ACK response.

**Root cause**: Docker inserts catch-all REJECT rules into both the INPUT and
FORWARD iptables chains. Non-Docker services (like the Python HTTP server used
for payload delivery) have no matching ACCEPT rule above the REJECT, so their
inbound traffic is silently dropped even though the port is listening.

```
# The blocking rules (Docker-managed):
Chain INPUT: REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
Chain FORWARD: REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
```

**Resolution**: Insert explicit ACCEPT rules before the REJECT rules:
```bash
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 9999 -j ACCEPT
sudo iptables -I FORWARD -p tcp --dport 9999 -j ACCEPT
```

**Key lesson**: On a Docker host, any non-Docker service needs explicit iptables
INPUT ACCEPT rules even when ufw is inactive. Docker manages iptables independently
of ufw.

## Issue 5 — Windows Server boot failure after ISO attach/detach

**Symptom**: After attaching and detaching Windows Server ISO multiple times during
a password recovery procedure, UEFI boot entries became corrupted. The firmware
found the hard disk but immediately bounced back to the boot menu.

**Resolution**: Boot from ISO, go to Recovery > Troubleshoot > Advanced Options >
Command Prompt, then rebuild the boot configuration:
```
bcdboot D:\Windows /s D: /f UEFI
```

Note: In the recovery environment, the Windows partition was assigned drive letter
`D:`, not `C:` — always verify with `diskpart > list volume` before running bcdboot.
