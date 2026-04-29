# JVLOT Texas County Registry — Session Log

Daily log for the recurring registry-discovery mission. Each session appends below.

---

## 2026-04-26 — Day 1 (Tier A: top metros)

**Foundation bootstrapped.** Working folder was empty at session start (no prior `countyRegistry.json`, `texas-counties-priority.txt`, `field-labels-discovered.json`, or `session-log.md`). Confirmed with the user that today is Day 1; created all four files from scratch. Priority list seeded with all 254 Texas counties in Tier A → B → C → D order per the mission spec.

**Counties processed today (12):**

- Bexar — `arcgis-found` — maps.bexar.org/arcgis/rest/services/Parcels/MapServer/0
- Harris — `arcgis-found` — gis.hctx.net/arcgis/rest/services/HCAD/Parcels/MapServer/0
- Travis — `arcgis-found` — gis.traviscountytx.gov/server1/rest/services/Boundaries_and_Jurisdictions/TCAD_public/MapServer/0
- Dallas — `arcgis-found` — maps.dcad.org/prdwa/rest/services/Property/ParcelQuery/MapServer/4 (ParcelPublishing)
- Tarrant — `arcgis-found` — mapit.tarrantcounty.com/arcgis/rest/services/Tax/TCProperty/MapServer/0
- Collin — `arcgis-found` — gismaps.cityofallen.org/arcgis/rest/services/ReferenceData/Collin_County_Appraisal_District_Parcels/MapServer/1 (note: layer 1, not 0)
- Denton — `arcgis-found` — geo.dentoncad.com/arcgis/rest/services/LinkedParcels2026/MapServer/3 (current service; older Public/2021_Parcels is 404)
- Williamson — `arcgis-found` — gis.wilco.org/arcgis/rest/services/public/county_wcad_parcels/MapServer/0
- Fort Bend — `arcgis-found` — gisweb.fortbendcountytx.gov/arcgis/rest/services/InteractiveMap/Parcels/MapServer/1 (note: layer 1, not 0; CAD's own gis.fbcad.org/serverarcgis2 was unreachable)
- Galveston — `needs-retry` — Parcels_VUEWorks_Version service confirmed via parent folder listing, but layer-level field metadata couldn't be loaded (silent navigation failure / response too large). URL pattern recorded for retry.
- Brazoria — `arcgis-found` — maps.brazoriacountytx.gov/arcgis/rest/services/general/Parcels/MapServer/1 (Parcel Information; layer 0 is Flagged Parcels only)
- Montgomery — `arcgis-found` — services6.arcgis.com/EbVsqZ18sv1kVJ3k/ArcGIS/rest/services/Montgomery_County_Parcels/FeatureServer/0

**Score:** 11 verified, 1 needs-retry, 0 realie-fallback, 0 unreachable, 0 blocked.

**Counties remaining:** 254 − 12 = 242 (of which 1, Galveston, will be retried tomorrow before moving to new counties).

**Field-labels added:** ~155 distinct field-name entries spanning property, owner, value, address, physical, legal, and system categories. This is a curated subset focused on JVLOT's display use case (owner / address / value / acreage / year built / legal). Each county's complete field list is preserved verbatim in `countyRegistry.json` so nothing is lost — `field-labels-discovered.json` is the semantic dictionary used by the display layer, not an exhaustive registry.

**Patterns observed:**

1. **No two CADs use the same schema.** Even semantically identical fields (owner name, total value, acres) have different names per county — `Owner` / `OWNER_NAME` / `owner_name_1` / `OWNERNME1` / `name` / `py_owner_name` etc. JVLOT will need a mapping layer per county, not a unified schema.
2. **Many CADs don't host their own ArcGIS — the county does.** Bexar (maps.bexar.org), Travis (gis.traviscountytx.gov), Tarrant (mapit.tarrantcounty.com), Williamson (gis.wilco.org), Brazoria (maps.brazoriacountytx.gov), Fort Bend (gisweb.fortbendcountytx.gov), Galveston (gis.galvestontx.gov), and Collin (gismaps.cityofallen.org!) all host parcels on the county or city government server, with copyright credit to the CAD. Only Harris, Denton, Dallas, and Montgomery use a CAD-branded host.
3. **Layer 0 is not always parcels.** In Brazoria, layer 0 is "Flagged Parcels" (permit holds) and the real layer is 1; in Collin it's also layer 1; in Denton it's layer 3; in Fort Bend it's layer 1 (and /0 returns 500). Always probe the MapServer root first.
4. **Some CADs have stale/test-named services.** Bexar's parcels-layer copyright says "Last update September 2019". Galveston's discoverable parcels endpoint is in a `VUEWorks_Test` folder (a sibling production folder exists). Williamson's description text says "Current as of 07/20/2015" but the service itself is live — note the description vs. real freshness.
5. **Montgomery's schema looks NY-style, not Texas-style.** Fields like SBL, SWIS, MUNI, VLG_TAXABL are characteristic of New York State assessor data — likely the parcel layer was provisioned from a generic template and the field names weren't relabeled. Worth a manual spot-check before going live.
6. **Egress is a constraint.** This session's bash sandbox returns 403 for all CAD URLs and `web_fetch` is allowlisted to only npm/pypi/github. Endpoint verification went through the Chrome MCP (navigate + get_page_text on `?f=json` / `?f=pjson`). Some servers (Galveston, FBCAD's own server) silently dropped Chrome's requests — likely IP-based rate-limiting or geo-blocking from Anthropic's egress IPs. Any future automation against these endpoints will need a dedicated egress allowlist or proxy.

**Estimated days remaining at this pace:** 12 counties/day × ~21 working days = full registry by ~mid May 2026. Tier B (Hill Country) tomorrow should go faster — smaller counties tend to outsource to BIS/TrueAutomation/Pritchard & Abbott shared portals which I expect to recognize as patterns once I see a few.

**Open questions for the user:**

- For Galveston, the only parcels endpoint I can find sits in a `VUEWorks_Test` (or `VUEWorks_Production`) folder hosted by the *City* of Galveston, not Galveston CAD. Is that acceptable, or do you want me to fall back to `realie-fallback` if I can't get a CAD-direct endpoint on tomorrow's retry?
- Several CAD endpoints have layers that include only a subset of fields the CAD actually publishes (e.g., Travis's TCAD_public layer has no value or owner fields — only address/legal/acres). For JVLOT to display values for those counties, a Realie or scraped-CAD fallback may be needed even where ArcGIS is "available". Flag for product-side decision once we see how many counties this affects.

---

## 2026-04-29 — Day 2 (Galveston retry + Tier B Hill Country)

**Working folder migrated.** Per user direction at start of session, all working files were moved from `C:\Users\justi\OneDrive\Desktop\CLAUDE PROJECTS\CLAUDE CO WORK\JVLOT\IDEATION` to `C:\Users\justi\OneDrive\Desktop\CLAUDE PROJECTS\CLAUDE CO WORK\JVLOT\GIS URLS`. The IDEATION versions are now stale; future sessions should pull from GIS URLS.

**File-state notes.** When the session began, `countyRegistry.json` and `field-labels-discovered.json` showed as 3 bytes via the bash mount (likely OneDrive Files-On-Demand stub state). The Read tool returned full content but during the recovery flow `countyRegistry.json` was rewritten with a partial reconstruction sourced from `lovable-counties-config.json`'s fieldMap — Tier A `fields` arrays are now the canonical-mapped subset rather than the original full raw ArcGIS field list. The lovable file remains untouched and authoritative for production. `field-labels-discovered.json` was preserved intact.

**Counties processed today (13 actions):**

- Galveston — `arcgis-found` (retry success) — gis.galvestontx.gov/server/rest/services/VUEWorks_Production/Parcels_VUEWorks_Version/MapServer/**29** (Day 1 had wrong layer: 0; correct layer is 29). Schema includes 2023-vintage value fields (VAL23TOT/VAL23LAND/VAL23IMP).
- Hays — `arcgis-found` — utility.arcgis.com/.../HaysCADWebService1/FeatureServer/0 (BIS Consulting)
- Comal — `arcgis-found` — utility.arcgis.com/.../ComalCADWebService/FeatureServer/0 (BIS)
- Kendall — `arcgis-found` — utility.arcgis.com/.../KendallCADWebService/FeatureServer/0 (BIS)
- Burnet — `arcgis-found` — utility.arcgis.com/.../BurnetCADWebService1/FeatureServer/0 (BIS, wkid 102739)
- Blanco — `arcgis-found` — utility.arcgis.com/.../BlancoCADWebService/FeatureServer/0 (BIS, wkid 102739)
- Llano — `arcgis-found` — utility.arcgis.com/.../LlanoCADWebService/FeatureServer/0 (BIS, wkid 102739)
- Bandera — `arcgis-found` — utility.arcgis.com/.../BanderaCADWebService/FeatureServer/0 (BIS) — real CAD is bancad.org, NOT banderacad.org (which is a third-party private search portal)
- Gillespie — `arcgis-found` — utility.arcgis.com/.../GillespieCADWebService/FeatureServer/0 (BIS, wkid 102739)
- Kerr — `arcgis-found` — utility.arcgis.com/.../KerrCADWebService/FeatureServer/0 (BIS)
- Medina — `needs-retry` — utility.arcgis.com/.../MedinaCADWebService/FeatureServer/0 (BIS) — endpoint identified and schema confirmed via in-app fetch, but direct fetch returns HTTP 403 (referer-locked). JVLOT may need a server-side proxy.
- Atascosa — `arcgis-found` — utility.arcgis.com/.../AtascosaCADWebService/FeatureServer/0 (BIS)
- Wilson — `arcgis-found` — utility.arcgis.com/.../WilsonCADWebService/FeatureServer/0 (BIS) — real domain is wilson-cad.org (with hyphen); www.wilsoncad.org DNS-fails

**Score:** 12 verified (Galveston + 11 Tier B), 1 needs-retry (Medina), 0 realie-fallback, 0 unreachable, 0 blocked.

**Counties remaining:** 254 − 12 (Tier A, excluding the Galveston retry) − 12 (Tier B today) = 230 (of which Medina is to be re-attempted from a different access path).

**Field-labels added:** ~40 new field-name entries — almost all from the BIS Consulting standard schema (`prop_id_text`, `file_as_name`, `addr_line1/2/3`, `addr_city/state`, `zip`, `situs_num/street/prefx/sufix/city/state/zip`, `legal_desc/2/3`, `land_val`, `imprv_val`, `market`, `legal_acreage`, `hood_cd`, `school`, `Deed_Date/Seq`, `tract_or_lot`, `abs_subdv_cd`, `next_appraisal_dt`, `map_id`, `ObjectID_1`) plus Galveston's distinct schema (`GEOID`, `LEGAL`, `ENTITIES`, `EXEMPT`, `LANDUSE`, `NBHD`, `VAL23TOT/LAND/IMP`, `ADDRESS/2/3`, `SITUS_NO`, etc.).

**Patterns observed:**

1. **All 12 Tier B counties use BIS Consulting.** The Hill Country is dominated by a single ArcGIS Online vendor (BIS, b. https://gis.bisclient.com/<countyname>cad/). Their parcels layer is layer 0, geometry is `Shape`, ObjectID is `ObjectID_1`, and the standard schema is the same across counties (~44-48 fields). This means one-shot ingestion logic can cover all 12 — JVLOT only needs different `queryUrl`s.
2. **BIS standard schema is sparse.** No `year_built`, `bedrooms`, `bathrooms`, or `living_area_sqft` are exposed. Hill Country users care most about acreage and value, which BIS does provide — but JVLOT will need a Realie supplement to display improvement characteristics.
3. **The proxy URL pattern is `utility.arcgis.com/usrsvcs/servers/<orgID>/rest/services/<county>CADWebService[1]/FeatureServer`.** Each county has a distinct 32-char hex `orgID`. Some counties append "1" (e.g., HaysCADWebService**1**, BurnetCADWebService**1**) and most don't — capture the exact name from network traffic each time, don't guess.
4. **spatialReference varies between 102740 and 102739** (NAD 1983 StatePlane Texas South Central vs Texas Central, in feet). Map projection logic in JVLOT must handle both per-county.
5. **Some CADs use third-party "free public search" sites that look like CADs but aren't.** Bandera was the gotcha today: `banderacad.org` is a private third-party search portal sponsored by `whoownsit.com`; the real Bandera CAD is `bancad.org`. Watch for this pattern in remaining tiers.
6. **Two domain-name oddities to note:** (a) Wilson CAD's `www.wilsoncad.org` returns DNS error — the working domain is `wilson-cad.org` (with hyphen); (b) Burnet CAD is `burnet-cad.org` (also hyphenated; `www.burnetad.org` is a separate placeholder).
7. **One county is referer-locked.** Medina returns HTTP 403 from a clean fetch but works inside the BIS app context. Investigate before launch — may need a server-side proxy, or it may be temporary tightening that resolves on retry.
8. **Galveston Day 1 layer ID was wrong.** The Day 1 entry pointed to layer 0; the actual parcels layer is **29**. Lesson: always probe MapServer/FeatureServer root with `?f=json` to enumerate layers — never assume layer 0.

**Estimated days remaining at this pace:** 12 quality counties/day × ~21 working days from here = ~mid-to-late May 2026 to complete all 254. Tier C tomorrow (Bell, El Paso, Nueces, Cameron, Hidalgo, Lubbock, McLennan, Webb, Smith, Ector, Midland, Brazos) — these are mid-size metros, expect a mix of BIS / county-government-hosted / dedicated CAD ArcGIS servers.

**Open questions for the user:**

- **Medina referer lock:** the endpoint URL is correct but blocks direct fetches. Three options: (a) keep `needs-retry` and try a different access path on Day 3, (b) mark as `realie-fallback` and not use ArcGIS for Medina, (c) accept it and have JVLOT proxy Medina requests through a server. Recommendation: leave as `needs-retry` for now — it's still a small enough share of statewide that we can decide later.
- **Galveston 2023 values:** the ArcGIS layer's `VAL23*` fields look like 2023 vintage. GCAD's data refresh schedule is ~Aug/Sept each year. JVLOT may want to surface a "data as of" badge for Galveston so users know the values aren't current-year.
- **BIS counties have no improvement data.** Realie supplement plan: confirm scope with the product side once we see how many counties total fall into "no year_built / no living_area" — likely all of Tier B and probably a chunk of Tier D too.

---
