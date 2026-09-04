# Network Setup

Lab networking configuration for a 4-VM VirtualBox environment: `linuxt` (monitored
endpoint), `theanalyst` (Elastic Stack host), `Kali` (attack box), and a Windows Server
endpoint. Each Linux VM runs two adapters — NAT (internet/updates) and Host-Only
(isolated inter-VM lab segment).

## What's documented here

- **netplan-fix.md** — root cause and fix for a downed `enp0s8` interface: the netplan
  config had no `ethernets:` section at all, then a follow-up fix introduced a
  one-character typo (`enp0a8` vs `enp0s8`) that silently prevented the config from
  applying. Final working config included.
- **nat-vs-hostonly.md** — why two VMs on their own NAT adapters (both showing `10.0.2.x`)
  could not reach each other, and why the Host-Only adapter is the correct shared
  segment for inter-VM lab traffic.
- **dhcp-hostonly-fix.md** — the lab's Host-Only network had its DHCP server disabled by
  default in VirtualBox's Host Network Manager. Documents enabling it and the resulting
  address scheme (`192.168.56.0/24`).
- **kali-adapter-fix.md** — Kali initially only had a NAT adapter and no path to the
  Host-Only segment; documents adding the second adapter.

## Screenshots referenced

See `/screenshots/` — filenames prefixed `network-` correspond to this section.
