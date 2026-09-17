# RDP Brute-Force Detection Rule

## Rule details

- **Type**: Custom query
- **Name**: RDP Brute Force - Failed Logon Attempts
- **Index**: `logs-system.security*`
- **Query**: `event.code: "4625" and host.os.type: "windows"`
- **Severity**: Medium
- **Risk score**: 47

## Why `host.os.type` instead of `agent.name`

Agent names change every time the Elastic Agent is re-enrolled (which happens
frequently in a snapshot-based lab environment). Using `host.os.type: "windows"`
as the Windows endpoint filter means the rule continues to work correctly after
any number of re-enrollments without needing to update the rule's query.

## How it was validated

1. Configured Windows Server for brute-force tool compatibility:
   - NLA (Network Level Authentication) disabled
   - Security Layer set to 0 (RDP, not TLS)

2. Ran ncrack against the Windows Server:
   ```bash
   ncrack -u Administrator -P wordlist.txt rdp://192.168.56.104
   ```

3. Failed logon events (Event ID 4625) confirmed in Kibana Discover

4. Detection rule alert confirmed firing in Security > Alerts

## Notes on tool compatibility

Hydra and Crowbar RDP modules did not generate 4625 events because they failed
to complete the RDP protocol handshake entirely — their connection attempts never
reached the authentication stage. ncrack's more complete RDP implementation
successfully completed the handshake and generated real failed logon events.

This is an important finding: in a real environment, an attacker using a more
capable tool would generate different telemetry than an attacker using a
less capable one. Detection rules should account for this.
