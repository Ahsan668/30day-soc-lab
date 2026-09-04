# Kali Missing Host-Only Adapter

## Problem
`ping 192.168.56.102` from Kali failed with 100% packet loss.

## Root cause
`ip a` on Kali showed only two interfaces: `lo` and `eth0` (NAT, `10.0.2.15/24`) — no
second adapter existed at all, so Kali had no path to the Host-Only segment.

## Fix
VirtualBox Manager -> Kali VM -> Settings -> Network -> Adapter 2 -> Enable Network
Adapter -> Attached to: Host-only Adapter -> same network name as `linuxt`/`theanalyst`.

After reboot, Kali received a `192.168.56.x` address via DHCP, and
`ping 192.168.56.102` succeeded.
