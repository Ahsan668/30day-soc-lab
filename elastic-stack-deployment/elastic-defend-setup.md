# Elastic Defend (EDR) Setup

## What is Elastic Defend?

Elastic Defend is Elastic's own Endpoint Detection and Response (EDR) solution.
It installs directly on endpoints as part of the Elastic Agent and provides:

- **Prevention** — actively blocks malware and malicious processes in real time
- **Detection** — generates detailed telemetry about threats independently of Sysmon
- **Response** — can isolate a compromised host remotely (requires trial/paid license)

The key distinction from Sysmon-based detection: Sysmon logs events passively for
later analysis. Elastic Defend actively intervenes to stop threats as they happen,
while simultaneously generating its own rich telemetry.

## Installation

1. Kibana → hamburger menu → **Management → Integrations**
2. Search for **"Elastic Defend"** → click **"Add Elastic Defend"**
3. Configure:
   - **Integration name**: `Elastic-Defend`
   - **Configuration**: Traditional Endpoints
   - **Protection level**: Complete EDR (requires 30-day trial)
4. Under "Where to add this integration" → **Existing hosts** → select `Windows-Endpoint`
5. Click **Save and continue** → **Save and deploy changes**

## Verification

Kibana → **Security → Manage → Endpoints** — Windows Server should appear with
Elastic Defend status showing **Healthy**.

Available actions from the Endpoints page:
- **Isolate host** — cuts machine off from the network (trial/paid only)
- **Respond** — opens a live response terminal on the endpoint
- **Release host** — restores network connectivity after isolation

## Testing

Running `servicehost.exe` (the Apollo C2 payload) on the Windows Server after
Elastic Defend installation resulted in:
- Immediate blocking — payload never executed
- Alert generated: **"Malware - Prevented - Elastic Endgame"** (High severity, risk 73)
- Alert visible in Kibana Security → Alerts within seconds

## Automated response action — host isolation

Configured the Malware Prevention rule to automatically isolate the host when malware
is detected:

1. Kibana → **Security → Rules** → search `malware`
2. Click **"Malware - Prevented - Elastic Endgame"**
3. **Edit rule settings** → scroll to **Response actions**
4. **Add response action** → select **Elastic Defend** → select **Isolate**
5. Save changes

**Effect**: when the Apollo payload was executed after this configuration, the Windows
Server was automatically isolated from the network — confirmed by 100% packet loss
from Kali (`ping 192.168.56.104`). The attacker's C2 connection dropped immediately
while the machine remained accessible via the Kibana Respond console for forensic
investigation.

**To release**: Security → Manage → Endpoints → Actions → Release host

## License note

- **Free tier**: prevention and detection work fully
- **Trial/paid**: host isolation, live response terminal, and advanced response actions
- This lab used the 30-day free trial to demonstrate host isolation

## Telemetry in Kibana

Elastic Defend generates its own events independently of Sysmon. Search in Discover:

```
event.category: "malware"
```

or

```
event.action: "malware_prevention"
```

These events include file hashes, process details, and the specific malware signature
that triggered the prevention — richer than Sysmon's generic process creation logs
for malware-specific events.
