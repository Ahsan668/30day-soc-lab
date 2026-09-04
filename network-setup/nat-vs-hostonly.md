# NAT Isolation vs Host-Only Networking

## The problem
`linuxt` and `Kali` both showed `10.0.2.x` NAT addresses, but pings and Hydra attempts
between them failed with connection refused / timeouts.

## Why
Each VM's NAT adapter is its own private, isolated network in VirtualBox — two VMs
showing the same `10.0.2.x`-style address are NOT on a shared network. NAT is
outbound-only by design (internet access for the VM), not inter-VM communication.

## The fix
Use the Host-Only adapter instead — a virtual network segment genuinely shared between
VMs attached to the same Host-Only network name (e.g. `vboxnet0`). All lab VMs needed:

- Adapter 1: NAT (internet/updates)
- Adapter 2: Host-only Adapter, same network name across all VMs

Final shared segment used: `192.168.56.0/24`.
