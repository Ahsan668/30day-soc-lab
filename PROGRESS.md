# Progress Log

Notes covering the MyDFIR 30-Day SOC Analyst Challenge. Built entirely on
local hardware using VirtualBox rather than a cloud platform (Vultr, as the standard
challenge suggests) — a deliberate deviation that meant handling all VM networking,
DHCP, and inter-VM connectivity manually rather than relying on a cloud provider's
pre-configured networking.

## Logical architecture diagram

- Drew the lab's logical architecture diagram in draw.io/diagrams.net before building
  anything, mapping out how the VMs, agents, and SIEM components would connect.

## ELK Stack introduction

- Covered ELK Stack fundamentals: log parsing, log types, visualization concepts, and
  filtering, as introduced in the standard challenge material.

## Environment deployment

- Deployed the core environment entirely on local VirtualBox rather than a cloud
  platform (Vultr):
  - Installed Elasticsearch, Kibana, and Fleet Server — all on a single VM
    (`theanalyst`) — to centralize log storage, visualization, and agent management.
  - Downloaded and installed Windows Server 2022 as a target/monitored endpoint.
  - Downloaded and installed Ubuntu Server as the Linux target/monitored endpoint
    (`linuxt`).
  - Installed and configured Sysmon on the Windows Server VM, using the default
    Sysmon configuration (later upgraded to SwiftOnSecurity config).
  - Enrolled endpoints into the self-hosted Fleet Server.

## Brute-force introduction

- Covered brute-force attack concepts, common tools, and defenses as part of the
  standard challenge material.

## First SSH brute-force attack

- Downloaded and installed Kali Linux as the dedicated attack VM (came with Hydra
  pre-installed).
- Ran the first SSH brute-force attack from Kali against the lab's own Ubuntu server.

## Remote access and Elastic Agent troubleshooting

- Set up remote SSH access to the `linuxt` endpoint VM, working through two methods:
  VirtualBox NAT port forwarding and direct-IP access over a Host-Only network.
- Diagnosed and fixed a downed network interface (`enp0s8`), tracing it back to a
  missing/incorrect netplan configuration — including a one-character typo
  (`enp0a8` instead of `enp0s8`) that caused the config to silently do nothing.
- Found the lab's VirtualBox Host-Only network had DHCP disabled by default, enabled
  it, and resolved a duplicate-IP-lease issue with `ip addr flush`.
- Deployed Elastic Agent on `linuxt`, enrolling it into the self-hosted Fleet Server on
  `theanalyst`. Hit and resolved a self-signed certificate trust error
  (`x509: certificate signed by unknown authority`) using `--insecure`.
- Diagnosed why the agent showed enrolled-but-offline in the Fleet UI: had been using
  `elastic-agent enroll` (one-time handshake only) instead of `elastic-agent install`
  (which also registers the persistent systemd service).

## Attack validation and dashboarding

- Confirmed the full stack was healthy on `theanalyst`.
- Fixed a missing second network adapter on the Kali attack VM.
- Ran a simulated SSH brute-force attack from Kali against `linuxt` using Hydra.
- Verified failed SSH login events in Kibana Discover.
- Diagnosed GeoIP world-map panel showing no data — correctly identified that private
  RFC1918 source IPs have no public GeoIP mapping.
- Built and saved a "failed SSH logins over time" Lens visualization to a dashboard.
- Built a custom query detection rule in Kibana Security > Rules for SSH brute-force,
  confirmed generating real alerts on the Security > Alerts page.

## RDP concepts

- Learned what RDP is, how attackers discover exposed RDP endpoints via Shodan/Censys,
  and key mitigations: disabling RDP when not in use, enforcing MFA, strong/unique
  passwords, avoiding default accounts.

## Additional work — RDP brute-force

- Diagnosed NLA (Network Level Authentication) and SecurityLayer settings as blockers
  preventing automated RDP brute-force tools from completing authentication.
- Confirmed RDP fully functional via direct `xfreerdp` manual connection.
- Crowbar and Hydra RDP modules failed against Windows Server 2022's RDP stack —
  documented as a real tool-compatibility finding.
- `ncrack` successfully cracked the RDP credentials, confirming the target was correctly
  configured and the limitation was specific to Crowbar/Hydra's RDP implementations.
- Confirmed failed RDP logon events (Event ID 4625) in Kibana Discover.
- Built a custom query detection rule for RDP brute-force, confirmed generating alerts.

## Additional work — Resource troubleshooting

- Diagnosed a Kibana detection rule execution failure: 20-minute timeout caused by
  memory pressure on `theanalyst` (single VM running Elasticsearch + Kibana + Fleet
  Server with 0B swap). Fixed with clean service restart.
- Diagnosed repeated "previously unenrolled" Elastic Agent crash loop on Windows Server
  — caused by snapshot restores reverting agent enrollment state. Fixed by re-enrolling
  with fresh token each time. Documented as expected behavior after snapshot restores.
- Diagnosed Windows Server disk-full VM suspension events causing cascading failures
  across Fleet, agent, and RDP services — root cause was host laptop C: drive at 0.00GB
  free. Resolved by freeing space and relocating VM storage.
- Upgraded Sysmon from default config to SwiftOnSecurity config to enable Event IDs
  1, 3, 7, 11 and more (default config only captured Event IDs 1 and 5).

## Additional work — GeoIP test document

- Diagnosed GeoIP map showing no data for RDP attacks (same root cause as SSH: private
  RFC1918 source IPs have no GeoIP mapping).
- Manually inserted a test document with a real public IP (`8.8.8.8`) into the correct
  Elasticsearch data stream (`logs-system.security-default`) to prove the GeoIP
  pipeline works correctly when given a real public IP. United States appeared on the
  map as expected.

## Additional work — Mythic C2 server (Oracle Cloud)

- Deployed a new Ubuntu 22.04 VM on Oracle Cloud Free Tier (Always Free — Ampere A1
  shape, 1 OCPU/4GB RAM, Mumbai region) to host Mythic C2 separately from the local
  VirtualBox lab — necessary due to host laptop disk space constraints.
- Installed Docker, docker-compose, Make, and Git on the Oracle VM.
- Cloned Mythic repository and built the CLI binary.
- Installed Apollo agent and HTTP C2 profile.
- Configured Mythic to bind on all interfaces (not localhost-only) for external
  accessibility.
- Diagnosed and fixed Docker iptables FORWARD and INPUT chain REJECT rules blocking
  inbound connections to non-Docker ports — added explicit ACCEPT rules.
- Built an Apollo payload (WinExe format, callback host `http://141.148.194.147`,
  callback port 80, 10-second interval).
- Transferred payload to Windows Server via Python HTTP server.
- Executed payload on Windows Server — **confirmed active C2 callback in Mythic**.

## Additional work — C2 detection dashboards

- Built three Kibana detection dashboards covering the Apollo C2 attack chain:

  **Dashboard 1 — Process Creation (Sysmon Event ID 1 / Windows Sysmon)**
  Fields: `winlog.event_data.CommandLine`, `winlog.event_data.Image`,
  `winlog.event_data.ParentCommandLine`, `winlog.event_data.ParentImage`,
  `winlog.event_data.ProcessGUID`, `winlog.event_data.User`,
  `winlog.event_data.CurrentDirectory`
  Purpose: detect processes running from unusual paths (Apollo payload injecting into
  svchost.exe from a non-standard location).

  **Dashboard 2 — Network Connection (Sysmon Event ID 3 / Windows Sysmon)**
  Fields: `winlog.event_data.Initiated: true`, `winlog.event_data.Image`,
  `winlog.event_data.SourceIp`, `winlog.event_data.DestinationIp`,
  `winlog.event_data.DestinationPort`
  Purpose: detect outbound C2 callback connections initiated by processes.

  **Dashboard 3 — Windows Defender Disabled (Event ID 5001)**
  Fields: `@timestamp`, `host.name`, `winlog.event_data.ProductName`,
  `winlog.provider_name` (Windows Defender)
  Purpose: detect real-time protection being disabled — common post-exploitation
  defense evasion step.

- Together these three tables cover the full C2 kill chain: execution → C2
  communication → defense evasion.

## Not yet documented

Further work is ongoing and will be documented here as completed.
