# GeoIP Map Panel — Investigation

## Symptom
A Kibana Maps visualization (World Countries layer, joined on
`source.geo.country_iso_code`) showed no colored/highlighted countries, despite
confirmed failed-login events existing in Discover.

## Investigation
Checked the raw value of `source.geo.country_iso_code` on individual events in
Discover — the field was blank/missing on every matching document.

## Root cause
`source.geo.country_iso_code` is populated by a GeoIP lookup against the source IP.
GeoIP databases only contain entries for public, globally-routable IP addresses. The
attacking IP (`192.168.56.x`, Kali) is a private RFC 1918 address — private ranges are
not registered to any country, so there is nothing for the GeoIP processor to find.
This is expected behavior, not a broken pipeline.

## Conclusion
GeoIP world maps are the correct tool for **external/internet-facing** attack traffic
(e.g. real brute-force attempts against a public-facing server), not for
internal-to-internal lab traffic where source and destination are both private IPs.

## More appropriate panels for this lab's traffic
- Failed logins over time (line/bar chart) — see `/dashboards/`
- Top source IPs (table/bar chart)
- Top attempted usernames
- Source -> destination IP pairing table
