# Attack Simulation

Simulated an SSH brute-force attack from the Kali VM against the monitored endpoint
(`linuxt`, `192.168.56.102`) using Hydra, to generate real failed-authentication events
for the detection pipeline to capture.

## What's documented here

- **hydra-ssh-bruteforce.md** — commands used, wordlist notes/issues, and the network
  troubleshooting required before the attack could actually reach its target.
- **hydra-rdp-bruteforce.md** — RDP brute-force run against the Windows Server VM,
  confirming the Windows telemetry pipeline (Sysmon -> Elastic Agent) end to end.

## Screenshots referenced

See `/screenshots/` — filenames prefixed `attack-` correspond to this section.
