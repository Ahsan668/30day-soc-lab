# 30-Day SOC Analyst — Home Lab

A hands-on SOC analyst lab built entirely on local hardware using VirtualBox, following
the 30-Day SOC Analyst curriculum. All infrastructure runs locally
rather than on a cloud platform — an intentional constraint that added real networking
and hardware challenges throughout the build.

## What was built

- **SIEM**: Elasticsearch + Kibana + Fleet Server on a single Ubuntu VM (`theanalyst`)
- **Monitored endpoints**: Windows Server 2022 and Ubuntu Server 20.04 (`linuxt`)
- **Attack box**: Kali Linux
- **C2 server**: Mythic on Oracle Cloud Free Tier (Ubuntu 22.04, ARM)
- **Ticketing system**: osTicket on Oracle Cloud Free Tier (Ubuntu 22.04)
- **Agents**: Elastic Agent on both endpoints, enrolled into self-hosted Fleet Server
- **Sysmon**: SwiftOnSecurity config on Windows Server (Event IDs 1, 3, 5, 7, 11+)
- **EDR**: Elastic Defend on Windows Server (Complete EDR, 30-day trial)

## Lab network

| VM | Role | IP |
|---|---|---|
| `theanalyst` | Elasticsearch + Kibana + Fleet Server | `192.168.10.10` |
| `linuxt` | Ubuntu target endpoint | `192.168.56.102` |
| Windows Server 2022 | Windows target endpoint | `192.168.56.104` |
| Kali Linux | Attack box | `192.168.56.103` |
| `mythic-c2` (Oracle Cloud) | Mythic C2 server | `141.148.194.147` |
| `osticket-server` (Oracle Cloud) | osTicket ticketing system | `141.148.196.50` |

## What was accomplished

### Infrastructure
- Full ELK Stack deployed and configured locally (not cloud)
- Fleet Server managing two enrolled Elastic Agents
- Sysmon deployed with SwiftOnSecurity config on Windows Server
- Mythic C2 server deployed on Oracle Cloud Always Free tier
- osTicket deployed on Oracle Cloud Always Free tier
- Elastic Defend (EDR) installed on Windows Server endpoint

### Attack simulation
- SSH brute-force attack (Hydra) against Ubuntu endpoint — credentials cracked
- RDP brute-force attack (ncrack) against Windows Server — credentials cracked
  - Crowbar and Hydra RDP modules documented as incompatible with Windows Server 2022
    RDP stack — ncrack succeeded where they failed
- Mythic Apollo C2 payload built, delivered, and executed — active callback confirmed
- C2 investigation: traced attack chain using Sysmon Event IDs 1, 3, and 11

### Detection and monitoring
- SSH brute-force detection rule — confirmed firing alerts
- RDP brute-force detection rule (Event ID 4625) — confirmed firing alerts
- GeoIP world map dashboard — confirmed working with test public IP document
- C2 detection dashboard — three tables covering process creation (Event ID 1),
  network connections (Event ID 3), and Defender disabled (Event ID 5001)
- Elastic Defend malware prevention — confirmed blocking Apollo payload in real time
- Automated host isolation response action configured on malware prevention rule

### Ticketing system integration
- osTicket integrated with Kibana via webhook connector
- SSH and RDP detection rules automatically create osTicket tickets when alerts fire
- Tickets include alert name, severity, timestamp, and direct Kibana alert link
- Fixed osTicket API source code bug preventing IP-unrestricted API key validation

### Troubleshooting documented
- Elastic Agent "previously unenrolled" crash loop after snapshot restores
- Host disk-full events causing VirtualBox VM suspension cascades
- NLA and SecurityLayer blocking RDP brute-force tool compatibility
- Docker iptables FORWARD/INPUT REJECT rules blocking non-Docker port traffic
- Winlogbeat subprocess not spawning in Elastic Agent 8.11.x (known compatibility issue)
- osTicket API `getApiKey()` returning raw string instead of database object (source
  code fix applied to `include/class.api.php`)

## Repository structure

```
30day-soc-lab/
├── README.md
├── PROGRESS.md
├── SCREENSHOT_CHECKLIST.md
├── diagrams/
│   ├── lab-architecture.drawio
│   └── mythic-attack-diagram.drawio
├── screenshots/
├── network-setup/
│   └── lab-network-config.md
├── elastic-stack-deployment/
│   ├── elk-stack-setup.md
│   └── resource-troubleshooting.md
├── attack-simulation/
│   ├── hydra-ssh-bruteforce.md
│   ├── hydra-rdp-bruteforce.md
│   └── mythic-c2-setup.md
├── detection-validation/
│   ├── detection-rule-ssh-bruteforce.md
│   └── detection-rule-rdp-bruteforce.md
└── dashboards/
    ├── geoip-dashboard.md
    └── c2-detection-dashboard.md
```

## Current status

Challenge complete. All major components built, tested, and documented.
