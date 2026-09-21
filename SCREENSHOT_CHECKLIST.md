# Screenshot Checklist

All screenshots are stored in the `screenshots/` folder.

## Architecture

- [x] Lab architecture diagram — `lab-architecture.png`
- [x] Mythic attack diagram — `mythic-attack-diagram.png`

## Elastic Stack

- [x] Kibana login page — `kibana-login.png`
- [x] Fleet > Agents showing endpoints Healthy — `elastic-fleet-online.png`
- [x] Elastic Agent systemctl status on theanalyst — `elastic-theanalyst-systemctl-status.png`

## SSH brute-force

- [x] Hydra attack command running — `attack-hydra-running.png`
- [x] Linux auth logs showing failed attempts — `attack-linux-auth-log.png`
- [x] Kibana Discover — failed SSH logon events — `detection-discover.png`
- [x] Kibana Discover query — `detection-discover-query.png`
- [x] Dashboard failed login timeline — `dashboard-failed-login-timeline.png`
- [x] Dashboard full view — `dashboard-full-view.png`

## RDP brute-force

- [x] ncrack output showing credentials discovered — `rdp-bruteforce-ncrack.png`
- [x] xfreerdp successful manual connection — `rdp-xfreerdp-connected.png`
- [x] Kibana Discover — Event ID 4625 events — `rdp-failed-logons-discover.png`
- [x] Kibana Security Alerts — RDP rule firing — `rdp-detection-alert.png`

## GeoIP dashboard

- [x] GeoIP empty map (private IPs have no mapping) — `detection-geoip-empty-map.png`
- [x] GeoIP field blank — `detection-geoip-field-blank.png`
- [x] GeoIP map with USA highlighted (test document) — `geoip-map-usa.png`

## Network setup

- [x] Host-only DHCP settings — `network-host-only-dhcp-settings.png`
- [x] ip a output — `network-ip-a.png`
- [x] Kali adapter settings — `network-kali-adapter-settings.png`
- [x] Kali ip a — `network-kali-ip-a.png`
- [x] Netplan current config — `network-netplan-current.png`

## Mythic C2

- [x] Mythic Apollo agent installed — `mythic-agent-apollo.png`
- [x] Mythic HTTP C2 profile installed — `mythic-c2-http.png`
- [x] Mythic Payloads page — servicehost.exe built — `mythic-payload-servicehost.png`
- [x] Mythic Callbacks page — active Apollo callback — `mythic-active-callback.png`
- [x] Oracle Cloud Console — mythic-c2 Running — `oracle-mythic-instance.png`

## C2 Detection Dashboards

- [x] Dashboard 1 — Process Creation table (Event ID 1) — `dashboard-process-creation.png`
- [x] Dashboard 2 — Network Connection table (Event ID 3) — `dashboard-network-connection.png`
- [x] Dashboard 3 — Defender Disabled table (Event ID 5001) — `dashboard-defender-disabled.png`

## osTicket

- [ ] osTicket staff portal login page — `osticket-staff-portal.png`
- [ ] osTicket ticket created by Kibana alert — `osticket-ticket-created.png`
- [ ] Kibana webhook connector configured — `kibana-osticket-connector.png`

## Elastic Defend

- [ ] Elastic Defend endpoint showing Healthy — `elastic-defend-endpoint-healthy.png`
- [ ] Malware Prevention alert in Kibana — `elastic-defend-malware-alert.png`
- [ ] Host isolation confirmed (packet loss from Kali) — `elastic-defend-host-isolated.png`
