# Screenshot Checklist

What to capture, when, how to name it, and where it's referenced. Take these as you
redo/verify each step in your lab (or from your existing terminal history/Kibana
session if you still have it open) — you don't need to redo the whole build, just
capture the current state of each piece.

Naming pattern: `<prefix>-<short-description>.png`, saved into `/screenshots/`.

## Network Setup (`network-` prefix)

| # | Screenshot | When to take it |
|---|---|---|
| 1 | `network-ip-a-before-fix.png` | Terminal showing `enp0s8` in `state DOWN`, no `inet` line — the broken state |
| 2 | `network-netplan-typo.png` | The `cat /etc/netplan/00-installer-config.yaml` output showing the `enp0a8` typo (if you still have it, otherwise skip) |
| 3 | `network-netplan-fixed.png` | The corrected netplan YAML file open in `nano` or via `cat` |
| 4 | `network-ip-a-after-fix.png` | `ip a` showing `enp0s8` now `UP` with a real `192.168.56.x` address |
| 5 | `network-hostonly-dhcp-settings.png` | VirtualBox Host Network Manager screen showing DHCP Server now enabled |
| 6 | `network-kali-adapter-settings.png` | Kali's VM Settings -> Network -> Adapter 2 showing Host-only Adapter enabled |

Referenced from: `/network-setup/README.md`

## Elastic Stack Deployment (`elastic-` prefix)

| # | Screenshot | When to take it |
|---|---|---|
| 7 | `elastic-fleet-offline.png` | Fleet UI showing `linuxt` agent as offline (the bug state) |
| 8 | `elastic-cert-error.png` | Terminal showing the `x509: certificate signed by unknown authority` error |
| 9 | `elastic-install-success.png` | Terminal output of the successful `elastic-agent install` run |
| 10 | `elastic-systemctl-status.png` | `sudo systemctl status elastic-agent` showing `active (running)` |
| 11 | `elastic-fleet-online.png` | Fleet UI showing `linuxt` agent now healthy/online |

Referenced from: `/elastic-stack-deployment/README.md`

## Attack Simulation (`attack-` prefix)

| # | Screenshot | When to take it |
|---|---|---|
| 12 | `attack-hydra-command.png` | The Hydra command you ran, in the Kali terminal |
| 13 | `attack-hydra-running.png` | Hydra mid-run with `-V` showing live attempts |
| 14 | `attack-linuxt-auth-log.png` | `linuxt` terminal showing `Failed password for linuxt from 192.168.56.x` lines live via `tail -f /var/log/auth.log` |

Referenced from: `/attack-simulation/README.md`

## Detection Validation (`detection-` prefix)

| # | Screenshot | When to take it |
|---|---|---|
| 15 | `detection-discover-query.png` | Kibana Discover with the query `agent.name:"linuxt" and system.auth.ssh.event:"Failed"` entered, showing matching results and timestamps |
| 16 | `detection-geoip-empty-map.png` | The GeoIP map panel showing no colored countries (the finding, not a bug — worth keeping as evidence of the investigation) |
| 17 | `detection-geoip-field-blank.png` | A single expanded document in Discover showing `source.geo.country_iso_code` is blank |

Referenced from: `/detection-validation/README.md`

## Dashboards (`dashboard-` prefix)

| # | Screenshot | When to take it |
|---|---|---|
| 18 | `dashboard-failed-logins-timeline.png` | The finished Lens chart showing the spike during the attack window |
| 19 | `dashboard-full-view.png` | The complete saved dashboard with all panels together |

Referenced from: `/dashboards/README.md`

**Note:** items 18–19 depend on finishing the in-progress visualization first — see
`/dashboards/failed-ssh-logins-timeline.md` for status.

## How to take and place them

1. Take the screenshot (Windows: `Win + Shift + S` for a selection snip).
2. Save it directly with the exact filename above into your local repo's
   `/screenshots/` folder before committing.
3. Reference it in the relevant `.md` file using:
   ```markdown
   ![description](../screenshots/network-ip-a-after-fix.png)
   ```
   (adjust the relative path `../` based on which folder the `.md` file is in — files
   inside a subfolder need `../screenshots/`, the root `README.md` just needs
   `screenshots/`)
