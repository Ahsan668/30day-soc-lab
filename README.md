# MyDFIR 30-Day SOC Analyst Challenge — Home Lab

A hands-on SOC/DFIR home lab built while working through the MyDFIR 30-Day SOC Analyst
Challenge. The goal was to stand up a working detection pipeline from scratch — not just
follow steps, but actually diagnose and fix the networking, service, and configuration
problems that came up along the way, the same way a junior analyst would in a real
environment.

Built entirely on local hardware using VirtualBox, rather than the cloud platform (Vultr) the standard challenge suggests — meaning all VM networking, DHCP, and inter-VM connectivity had to be configured and troubleshot manually rather than relying on a cloud provider's pre-wired networking.

## Tech Stack

- **Elasticsearch** — log storage and indexing
- **Kibana** — visualization and Discover-based log querying
- **Fleet Server** — central agent management, self-hosted (self-signed TLS cert)
- **Elastic Agent** — endpoint log shipping, deployed on the monitored Linux endpoint
- **Ubuntu Linux** — monitored endpoint (`linuxt`) and Elastic Stack host (`theanalyst`)
- **Kali Linux** — attack simulation box
- **Windows Server 2022** — additional monitored endpoint, Elastic Agent enrolled
- **Sysmon** — installed on Windows Server (default configuration) for endpoint telemetry
- **draw.io / diagrams.net** — used for the lab's logical architecture diagram
- **VirtualBox** — multi-VM lab environment (NAT + Host-Only networking)
- **Hydra** — SSH brute-force simulation tool

## Repo Structure

```
├── README.md                    <- you are here
├── PROGRESS.md                  <- day-by-day log
├── network-setup/               <- netplan fixes, DHCP/adapter troubleshooting
├── elastic-stack-deployment/    <- Fleet Server + Elastic Agent install/enroll
├── attack-simulation/           <- Hydra brute-force commands and notes
├── detection-validation/        <- Discover queries, findings, GeoIP investigation
├── dashboards/                  <- Kibana visualization/dashboard exports
└── screenshots/                 <- supporting screenshots, referenced by each section
```

Each subfolder has its own README explaining what's in it.

## What This Demonstrates

**1. Built a multi-VM detection lab from the ground up on VirtualBox**
Configured a segmented lab network (NAT for internet/updates, Host-Only for isolated
inter-VM traffic) across four VMs, including diagnosing why VMs on paired NAT adapters
couldn't reach each other and correcting the architecture to use a shared Host-Only
segment instead.

**2. Diagnosed and resolved real Linux networking failures**
Root-caused a downed network interface to a missing netplan configuration, found and
fixed a one-character interface-name typo that was silently preventing configuration
from applying, and resolved a disabled DHCP server on the lab's Host-Only network that
was leaving hosts with no IPv4 address.

**3. Deployed and troubleshot a self-hosted Elastic Stack detection pipeline**
Installed and enrolled Elastic Agent into a self-hosted Fleet Server, resolved a
self-signed certificate trust failure (`x509: certificate signed by unknown authority`),
and diagnosed a misleading "enrolled but offline" state caused by using the wrong CLI
subcommand (`enroll` vs `install`) — the agent needs `install` to run as a persistent
systemd service, not just complete a one-time handshake.

**4. Simulated an SSH brute-force attack and validated end-to-end log visibility**
Used Hydra from a Kali attack box to generate failed SSH authentication attempts against
the monitored endpoint, then confirmed the events were correctly shipped, indexed, and
queryable in Kibana Discover, with accurate timestamps.

**5. Investigated a dashboard data gap and identified the correct root cause**
When a GeoIP world-map panel showed no data, diagnosed that the source IPs involved were
private/RFC1918 addresses with no public GeoIP mapping — rather than assuming the
pipeline was broken — and identified more appropriate visualization types for
internal-to-internal lab traffic.

**6. Extended detection validation to a Windows endpoint via RDP brute-force**
Ran an RDP brute-force simulation against the Windows Server 2022 endpoint using Hydra and confirmed failed logon events reached Kibana Discover, validating the Sysmon -> Elastic Agent -> Elasticsearch -> Kibana pipeline on Windows, not just Linux.

**7. Built a saved detection rule, not just an ad-hoc search**
Converted the manual Discover query used to validate SSH brute-force events into a
custom query rule under Kibana Security > Rules — the actual difference between
analyst-driven log searching and scheduled, automated detection.

## Status

This lab covers **Days 1-15** of the 30-day challenge, fully documented day by day in
`PROGRESS.md` — from initial architecture planning through Elastic Stack deployment,
endpoint enrollment, SSH/RDP brute-force simulation, attack validation in Kibana, and
RDP security concepts. Days 16-30 are not yet covered in this repo.

## Notes on Scope

This is a personal lab environment, not a production deployment. TLS certificate
verification was intentionally disabled (`--insecure`) for Fleet Server enrollment,
which is appropriate for an isolated lab with a self-signed cert but would not be
appropriate in production.
