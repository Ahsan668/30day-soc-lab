# DHCP Server Disabled on Host-Only Network

## Problem
After fixing the netplan typo, `enp0s8` came up (`UP`, `LOWER_UP`) but had no IPv4
address — only a self-assigned `inet6 fe80::` link-local address.

## Root cause
VirtualBox's Host-Only network ships with its built-in DHCP server **disabled by
default**. Unlike NAT (which always auto-runs a DHCP server), Host-Only assumes you
might want static IPs and leaves DHCP off.

Checked via: VirtualBox Manager -> Tools/File -> Host Network Manager -> DHCP Server tab
-> "Enable Server" was unchecked.

## Fix
Enabled the DHCP server on the Host-Only network (`vboxnet0`). Resulting address range:
`192.168.56.0/24` (VirtualBox's default Host-Only range).

On each VM:

```bash
sudo dhclient enp0s8
ip a | grep -A 4 "enp0s8:"
```

## Cleanup note
Running `dhclient` more than once without releasing the old lease stacked a duplicate
address (`secondary dynamic`) on the interface. Cleaned up with:

```bash
sudo ip addr flush dev enp0s8
sudo dhclient enp0s8
```

## Final confirmed addresses
- `linuxt`: `192.168.56.102/24`
