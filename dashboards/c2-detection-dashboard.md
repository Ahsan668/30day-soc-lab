# C2 Detection Dashboard

Three Kibana table visualizations built to detect and investigate Apollo C2 activity
on the Windows Server endpoint. Together they cover the full attack chain: execution,
C2 communication, and defense evasion.

## Dashboard 1 — Process Creation

**Filter**: `event.provider: "Microsoft-Windows-Sysmon"` and `winlog.event_id: 1`

**Fields shown in table**:
- `winlog.event_data.CommandLine`
- `winlog.event_data.Image`
- `winlog.event_data.ParentCommandLine`
- `winlog.event_data.ParentImage`
- `winlog.event_data.ProcessGUID`
- `winlog.event_data.User`
- `winlog.event_data.CurrentDirectory`

**What it detects**: Processes running from unusual or non-standard paths. The Apollo
C2 payload injects into svchost.exe but runs it from a path other than
C:\Windows\System32 — a classic IOC for process injection and living-off-the-land
techniques.

## Dashboard 2 — Network Connection

**Filter**: `event.provider: "Microsoft-Windows-Sysmon"` and `winlog.event_id: 3`
and `winlog.event_data.Initiated: true`

**Fields shown in table**:
- `winlog.event_data.Image`
- `winlog.event_data.SourceIp`
- `winlog.event_data.DestinationIp`
- `winlog.event_data.DestinationPort`

**What it detects**: Outbound network connections initiated by processes on the
endpoint. Initiated: true filters specifically to connections the process opened
itself (outbound C2 callbacks), not inbound connections. Apollo's callback to
141.148.194.147:80 would appear here.

## Dashboard 3 — Windows Defender Disabled

**Filter**: `event.code: "5001"` and `winlog.provider_name: "Microsoft-Windows-Windows Defender"`

**Fields shown in table**:
- `@timestamp`
- `host.name`
- `winlog.event_data.ProductName`
- `winlog.provider_name`

**What it detects**: Windows Defender real-time protection being disabled. Event ID
5001 fires whenever Defender's real-time protection is turned off. The ProductName
field confirms which specific Defender component reported the status change.

## Attack chain narrative

These three tables together tell the complete story of a C2 compromise:

1. Process Creation table: something executed from an unusual path
2. Network Connection table: that process called out to an external IP
3. Defender Disabled table: real-time protection was subsequently disabled

MITRE ATT&CK mapping:
- T1055 — Process Injection
- T1071.001 — Application Layer Protocol: Web Protocols (C2 over HTTP)
- T1562.001 — Impair Defenses: Disable or Modify Tools
