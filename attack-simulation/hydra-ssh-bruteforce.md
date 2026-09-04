# SSH Brute-Force Simulation — Hydra

## Tool
Hydra (pre-installed on Kali) — used to attempt SSH authentication against `linuxt`
using a wordlist of candidate passwords.

## Command

```bash
hydra -l linuxt -P <wordlist> ssh://192.168.56.102 -t 4 -V
```

- `-l linuxt` — target username
- `-P <wordlist>` — password list to try
- `ssh://192.168.56.102` — target service and IP (Host-Only segment address; see
  `/network-setup/nat-vs-hostonly.md` for why the NAT IP did not work)
- `-t 4` — 4 parallel threads (Hydra's own recommended limit for SSH)
- `-V` — verbose, shows each attempt live

## Wordlist notes
`/usr/share/wordlists/rockyou.txt.gz` (Kali's standard bundled wordlist) was not present
on this installation's default wordlist directory listing. A small custom wordlist was
used as a fallback for generating test events:

```bash
printf "password123\nadmin\nletmein\ntest1234\n" > /tmp/small.txt
```

**Open item:** whether `rockyou.txt` was ultimately located/installed
(`sudo apt install wordlists -y`) and used for the full run, or whether the custom list
was used throughout, was not confirmed — fill in based on what was actually used.

## Run duration
Approximately 40–45 minutes.

## Prerequisites that had to be fixed first
- Kali needed a second (Host-Only) network adapter added — see
  `/network-setup/kali-adapter-fix.md`.
- Target IP had to be the Host-Only address (`192.168.56.102`), not the NAT address
  (`10.0.2.15`) — NAT networks are isolated per-VM in VirtualBox.
