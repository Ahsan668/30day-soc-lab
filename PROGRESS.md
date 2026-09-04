# Progress Log

Day-by-day notes covering the portion of the MyDFIR 30-Day SOC Analyst Challenge
documented in this repo. Only days with confirmed, specific work are listed.

## Day 13

- Set up remote SSH access to the `linuxt` endpoint VM, working through two methods:
  VirtualBox NAT port forwarding and direct-IP access over a Host-Only network.
- Diagnosed and fixed a downed network interface (`enp0s8`), tracing it back to a
  missing/incorrect netplan configuration — including a one-character typo
  (`enp0a8` instead of `enp0s8`) that caused the config to silently do nothing.
- Found the lab's VirtualBox Host-Only network had DHCP disabled by default, enabled
  it, and resolved a duplicate-IP-lease issue with `ip addr flush`.
- Deployed Elastic Agent on `linuxt`, enrolling it into a self-hosted Fleet Server on
  `theanalyst`. Hit and resolved a self-signed certificate trust error
  (`x509: certificate signed by unknown authority`) using `--insecure`.
- Diagnosed why the agent showed enrolled-but-offline in the Fleet UI: had been using
  `elastic-agent enroll` (one-time handshake only) instead of `elastic-agent install`
  (which also registers the persistent systemd service). Fixed by re-running with
  `install`, confirmed via `systemctl status elastic-agent` showing `active (running)`.

## Day 14

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
  country mapping — not a pipeline failure. Started building a "failed SSH logins
  over time" line/bar chart as a more appropriate panel for internal lab traffic
  (in progress, not yet confirmed saved to a dashboard).

## Not yet documented

Days 1–12 and 15–30 of the challenge are not covered in this log.
