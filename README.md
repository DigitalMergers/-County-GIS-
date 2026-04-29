# JVLOT County Registry

This repo holds a programmatically-built registry of every Texas county appraisal district's public ArcGIS parcel endpoint, plus a canonical field-name mapping per county.

**Consumed by:** the JVLOT app (Lovable) at runtime. The app fetches `lovable-counties-config.json` directly from this repo's raw URL.

**Files:**
- `lovable-counties-config.json` — production config consumed by JVLOT (canonical schema + per-county fieldMap)
- `countyRegistry.json` — raw research registry with full ArcGIS field arrays per county
- `field-labels-discovered.json` — semantic dictionary of field-name aliases across counties
- `texas-counties-priority.txt` — county processing order (254 total)
- `session-log.md` — daily notes from the discovery agent
- `agent-mission-county-discovery.md` — agent's mission spec

**Status:** in progress. 24 of 254 counties registered as of 2026-04-29 (Tier A metros + Tier B Hill Country).

**Public raw URL for the production config:**
https://raw.githubusercontent.com/DigitalMergers/-County-GIS-/main/lovable-counties-config.json
