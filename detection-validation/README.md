# Detection Validation

Confirms the simulated attack traffic was correctly captured, shipped, and queryable
end-to-end in Kibana — and documents a diagnostic investigation into a dashboard panel
that initially appeared broken.

## What's documented here

- **discover-query-validation.md** — the Kibana Discover query used to confirm failed
  SSH login events from the Hydra run were indexed with correct timestamps.
- **geoip-map-investigation.md** — why a GeoIP world-map panel showed no data (private
  IP addresses have no public GeoIP mapping), and what panel types are actually
  appropriate for internal-to-internal lab traffic.

## Screenshots referenced

See `/screenshots/` — filenames prefixed `detection-` correspond to this section.
