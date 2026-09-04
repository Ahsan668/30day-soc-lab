# RDP Brute-Force Simulation — Hydra

## Tool
Hydra — the same tool used for the SSH brute-force simulation (see
`hydra-ssh-bruteforce.md`), pointed at RDP instead.

## Target
Windows Server 2022 VM, with Elastic Agent and Sysmon installed and enrolled into the
same Fleet Server as the rest of the lab.

## Result
Confirmed failed RDP logon events appeared in Kibana Discover, validating that Windows
telemetry (via Sysmon -> Elastic Agent -> Elasticsearch -> Kibana) was flowing
correctly, the same way the SSH auth log pipeline was validated on `linuxt`.

## Open items
- Exact Hydra command/flags used for the RDP run were not documented in detail —
  fill in the specific command if you want it recorded here.
- The specific Discover query/field used to confirm failed RDP logons (Windows
  equivalent of `system.auth.ssh.event:"Failed"` — likely a Windows Security Event ID
  4625 match) was not documented — fill in if you want it recorded.
