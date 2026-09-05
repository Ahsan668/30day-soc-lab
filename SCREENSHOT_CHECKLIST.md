# Screenshot Checklist

Realistic version — split into what's actually still capturable vs. what's genuinely
gone because the fix is already applied. Don't break working things just to
re-screenshot a bug; that's backwards. Prose in the `.md` files already documents the
"before" state and root cause — the screenshots are supporting evidence for the
"after"/current state, not required proof of every bug.

Naming pattern: `<prefix>-<short-description>.png`, saved into `/screenshots/`.

## Network Setup (`network-` prefix)

| # | Screenshot | Status |
|---|---|---|
| 1 | `network-ip-a-current.png` | **Take now** — `ip a` showing `enp0s8` currently `UP` with `192.168.56.x` address |
| 2 | `network-netplan-current.png` | **Take now** — `cat /etc/netplan/00-installer-config.yaml` showing the current, correct config |
| 3 | `network-hostonly-dhcp-settings.png` | **Take now** — VirtualBox Host Network Manager showing DHCP Server enabled (this is saved settings, still there) |
| 4 | `network-kali-adapter-settings.png` | **Take now** — Kali's VM Settings -> Network -> Adapter 2 showing Host-only Adapter (saved settings, still there) |
| — | ~~before-fix / typo screenshots~~ | **Skip** — already fixed, not worth breaking again for a photo. The root-cause writeup in `netplan-fix.md` already documents the exact broken config and error. |

## Elastic Stack Deployment (`elastic-` prefix)

| # | Screenshot | Status |
|---|---|---|
| 5 | `elastic-fleet-online.png` | **Take now** — Fleet UI showing `linuxt` agent currently healthy/online |
| 6 | `elastic-systemctl-status.png` | **Take now** — `sudo systemctl status elastic-agent` showing `active (running)` |
| — | ~~cert error / offline screenshots~~ | **Skip** — already fixed. `cert-trust-issue.md` and `agent-install-enroll.md` already document the exact error text. |

## Attack Simulation (`attack-` prefix)

| # | Screenshot | Status |
|---|---|---|
| 7 | `attack-hydra-command.png` | **Reproducible** — the Hydra command itself, typed in a terminal (doesn't need to actually run to screenshot the command) |
| 8 | `attack-hydra-running.png` | **Optional re-run** — if you want a live-action shot, a short (10-30 second) re-run against your own lab VM is fine; this isn't "redoing the challenge," it's a documentation photo of a technique you already proved works |
| 9 | `attack-linuxt-auth-log.png` | Same as above — pairs with a short re-run, or use existing scrollback if you kept any |

## Detection Validation (`detection-` prefix)

| # | Screenshot | Status |
|---|---|---|
| 10 | `detection-discover-query.png` | **Take now** — the failed-login events are permanently stored in Elasticsearch; query them anytime with the same Discover search |
| 11 | `detection-geoip-empty-map.png` | **Take now** — this isn't a bug that got fixed, it's permanent behavior (private IPs never get GeoIP data), so it looks the same every time you open it |
| 12 | `detection-geoip-field-blank.png` | **Take now** — same reasoning as above, permanently reproducible |

## Dashboards (`dashboard-` prefix)

| # | Screenshot | Status |
|---|---|---|
| 13 | `dashboard-failed-logins-timeline.png` | **Pending** — depends on finishing the in-progress Lens visualization first (see `dashboards/failed-ssh-logins-timeline.md`) |
| 14 | `dashboard-full-view.png` | **Pending** — same, once the dashboard is saved |

## How to take and place them

1. Take the screenshot (Windows: `Win + Shift + S` for a selection snip).
2. Save it with the exact filename above into your local repo's `/screenshots/` folder.
3. Reference it in the relevant `.md` file:
   ```markdown
   ![description](../screenshots/network-ip-a-current.png)
   ```
   (root-level `README.md` uses `screenshots/` without the `../`)

## Bottom line

You lost nothing by not having "before" screenshots. The written root-cause
explanations already in each `.md` file are what an interviewer actually cares about —
"here's the error, here's why it happened, here's the fix" reads just as credibly in
text as in a screenshot. Screenshots are supporting proof of the *current working
state*, not a requirement to document every historical bug photographically.
