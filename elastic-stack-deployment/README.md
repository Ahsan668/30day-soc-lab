# Elastic Stack Deployment

Documents deploying Elastic Agent on the monitored endpoint (`linuxt`) and enrolling it
into a self-hosted Fleet Server running on `theanalyst` (which also hosts Elasticsearch
and Kibana).

## What's documented here

- **agent-install-enroll.md** — the `enroll` vs `install` distinction that caused the
  agent to show "enrolled" but "offline" in Fleet, and the fix.
- **cert-trust-issue.md** — the self-signed certificate trust error and how it was
  resolved for a lab environment.
- **service-verification.md** — confirming all three services (`elastic-agent`,
  `kibana`, `elasticsearch`) were active and running.

## Screenshots referenced

See `/screenshots/` — filenames prefixed `elastic-` correspond to this section.
