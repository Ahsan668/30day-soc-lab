# Failed SSH Logins Over Time — Build Steps

## Goal
A time-series chart showing failed SSH login attempts, to visually reveal the spike
during the Hydra brute-force run.

## Steps (Kibana Lens)
1. Visualize Library -> Create visualization -> Lens
2. Data view: same one used in Discover (contains `system.auth.ssh.event` data)
3. X-axis: `@timestamp` (interval: Auto, or 1 minute)
4. Y-axis: Count of records
5. Query bar filter:
   ```
   agent.name: "linuxt" and system.auth.ssh.event: "Failed"
   ```
6. Save and return -> add to dashboard, titled "Failed SSH Logins Over Time"

## Status
**Confirmed complete.** Built and saved to a dashboard. Export the dashboard JSON
(Stack Management -> Saved Objects -> find it -> Export) into this folder alongside a
screenshot, per `SCREENSHOT_CHECKLIST.md`.
