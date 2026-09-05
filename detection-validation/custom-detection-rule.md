# Custom Query Detection Rule — SSH Brute-Force

## What was built
A custom query rule in Kibana under **Security > Rules**, mirroring the Discover
search used earlier to validate the SSH brute-force events:

```
agent.name: "linuxt" and system.auth.ssh.event: "Failed"
```

Unlike a plain Discover search (which requires an analyst to manually run it), a
saved rule runs on a schedule and generates an alert automatically when matching
events occur — this is the actual difference between "ad-hoc log searching" and
"detection engineering."

## Status
Rule created and saved. **Not yet confirmed** whether it has actually fired/triggered
on a live event (e.g. a fresh Hydra re-run) — worth testing and documenting the
resulting alert (with a screenshot of it firing) to fully prove the detection works
end-to-end, not just that the rule exists.

## Open items
- Rule name/ID as configured in Kibana
- Alert schedule/interval used (how often it checks)
- Whether a notification action (email, Slack, etc.) was configured, or if it's
  alert-only within the Kibana Security app
