# Progress Log

Day-by-day notes covering Days 1-15 of the MyDFIR 30-Day SOC Analyst Challenge. Built
entirely on local hardware using VirtualBox rather than a cloud platform (Vultr, as the
standard challenge suggests) — a deliberate deviation that meant handling all VM
networking, DHCP, and inter-VM connectivity manually rather than relying on a cloud
provider's pre-configured networking.

## Day 1 — Logical architecture diagram

- Drew the lab's logical architecture diagram in draw.io/diagrams.net before building
  anything, mapping out how the VMs, agents, and SIEM components would connect.

## Day 2 — ELK Stack introduction

- Covered ELK Stack fundamentals: log parsing, log types, visualization concepts, and
  filtering, as introduced in the standard challenge material.

## Days 3-10 — Environment deployment

- Deployed the core environment entirely on local VirtualBox rather than a cloud
  platform (Vultr):
  - Installed Elasticsearch, Kibana, and Fleet Server — all on a single VM
    (`theanalyst`) — to centralize log storage, visualization, and agent management.
  - Downloaded and installed Windows Server 2022 as a target/monitored endpoint.
  - Downloaded and installed Ubuntu Server as the Linux target/monitored endpoint
    (`linuxt`).
  - Installed and configured Sysmon on the Windows Server VM, using the default
    Sysmon configuration.
  - Enrolled endpoints into the self-hosted Fleet Server.

## Day 11 — Brute-force introduction

- Covered brute-force attack concepts, common tools, and defenses as part of the
  standard challenge material.

## Day 12 — First SSH brute-force attack

- Downloaded and installed Kali Linux as the dedicated attack VM (came with Hydra
  pre-installed).
- Ran the first SSH brute-force attack from Kali against the lab's own Ubuntu server.

## Day 13 — Remote access and Elastic Agent troubleshooting

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
  (which also registers the persistent systemd service). Fixed by re-running with
  `install`, confirmed via `systemctl status elastic-agent` showing `active (running)`.

## Day 14 — Attack validation and dashboarding

- Confirmed the full stack was healthy on `theanalyst`: `elastic-agent.service`,
  `kibana.service`, and `elasticsearch.service` all `active (running)`.
- Fixed a missing second network adapter on the Kali attack VM (it only had a NAT
  adapter; added a Host-Only adapter to reach the shared lab segment).
- Ran a simulated SSH brute-force attack from Kali against `linuxt`
  (`192.168.56.102`) using Hydra, generating failed authentication events.
- Verified the attack traffic was captured end-to-end: confirmed failed SSH login
  events in Kibana Discover, filtered by `agent.name:"linuxt"` and
  `system.auth.ssh.event:"Failed"`, with correct timestamps matching the attack window.
- Attempted a GeoIP world-map dashboard panel; diagnosed that it showed no data
  because the source IP involved was a private/RFC1918 address with no public GeoIP
  country mapping — not a pipeline failure. Built a "failed SSH logins over time"
  line/bar chart instead, as a more appropriate panel for internal lab traffic —
  confirmed saved to a dashboard.
- Built a custom query detection rule in Kibana (Security > Rules), mirroring the
  Discover search used to validate the SSH brute-force events, saved as an alert
  rather than an ad-hoc search. Whether the rule has actually been confirmed to fire
  on a live re-run has not been verified yet.

## Additional work (day not specified)

- Ran an RDP brute-force attack against the Windows Server 2022 VM using Hydra, the
  RDP equivalent of the Day 12/14 SSH work.
- Confirmed failed RDP logon events appeared in Kibana Discover, validating that
  Windows Server telemetry (via Sysmon/Elastic Agent) was flowing into the stack
  correctly.

## Day 15 — RDP concepts

- Learned what RDP (Remote Desktop Protocol) is, how attackers commonly discover
  exposed RDP endpoints (Shodan, Censys), and the key mitigations: disabling RDP when
  not in use, enforcing MFA, using strong/unique passwords, and avoiding default
  accounts. See `attack-simulation/hydra-rdp-bruteforce.md` for full notes, tied
  directly to the RDP brute-force simulation already performed.

## Not yet done

- The custom query detection rule (Security > Rules) has been created, but it has not
  been confirmed to actually trigger/fire on a live re-run of the attack — worth
  verifying and documenting the fire event before calling this fully proven end-to-end.

## Not yet documented

Days 16-30 are not covered — this repo currently documents Days 1-15 only.
