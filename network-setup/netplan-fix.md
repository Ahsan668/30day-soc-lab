# netplan Fix — enp0s8 Interface Down

## Problem
`enp0s8` showed `state DOWN`, no IPv4 address, while `enp0s3` (NAT) worked fine.

## Root cause
`/etc/netplan/00-installer-config.yaml` contained only:

```yaml
network:
  version: 2
```

No `ethernets:` section — no interface was ever told to request DHCP or come up.

## First fix attempt (had a bug)

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0a8:        # <-- typo: should be enp0s8
      dhcp4: true
```

netplan did not error — it just had no effect on the real `enp0s8` interface, since
`enp0a8` doesn't exist.

## Corrected, working config

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
```

Applied with:

```bash
sudo netplan apply
```

## Verification

```bash
ip a | grep -A 4 "enp0s8:"
```

Confirmed `state UP` and (after the DHCP fix in `dhcp-hostonly-fix.md`) a real
`192.168.56.x` address.
