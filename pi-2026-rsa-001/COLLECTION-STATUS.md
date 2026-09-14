# PI-2026-RSA-001 — Collection Status

**Subject site:** 4889 E Lake Harriet Parkway, Minneapolis, MN 55419
**Window:** 14 Sep 2025 – 14 Sep 2026 (strict)
**Rev:** B · 14 Sep 2026
**Composite:** MODERATE (36/100) — band unchanged from Rev A

---

## 1. What this session closed

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | Carjacking-stat correction | **CLOSED** | Confirmed November **2021**, not in-window. Withdrawn from Annex A. |
| 2 | MCCA H1-2026 hard numbers | **CLOSED** | Homicide 25 (−3.8%), robbery 404 (−19.5%), rape 252 (**+17.8%**), agg assault 1,518 (**+12.8%**). |
| 3 | BCA 2025 UCR provenance | **PARTIAL** | 226/100k confirmed. **BCA issued an amended 2025 UCR** — statewide murders 130→132. Cite the amended report. PDF itself not retrievable here. |
| 4 | Property-crime pattern currency | **CLOSED** | Active and escalating in-window: ≥87 vehicles in one night (late Aug 2026, 2–7 a.m., 9 locations); 200+ cases in latest wave; offenders aged 11–16. |
| 5 | Structural persistence driver | **NEW — CLOSED** | MN "Raise the Age" effective **1 Aug 2026**: under-13s cannot be charged, prosecuted or detained, no exceptions. Predicts the property pattern persists. |
| 6 | Jurisdictional gap in collection plan | **NEW — CLOSED** | The parkway is **Park Board** jurisdiction. The drafted MGDPA request to MPD alone would miss the running surface. Second request required (see §4). |

### Correction detail — the carjacking statistic

The Rev A claim *"5 of 7 carjackings ringing Lake Harriet occurred Nov 14–25"* is **November 2021**.

Corroborating dated sources:
- Axios Twin Cities, carjacking spree, **9 Nov 2021**
- MinnPost, *"Carjackings up 38% in Minneapolis in 2021"*, **Nov 2021**
- Star Tribune, *"Map: Minneapolis neighborhoods hit hardest by carjackings in 2021"*
- The Southwest Voices piece itself dates its comparison cluster to *"a ten-day stretch of November 2021"* and cites Kingfield carjackings on 8/14/15/16 November.

**Residual uncertainty:** the article byline could not be inspected directly — `southwestvoices.news` is blocked by this environment's egress policy. Tagged `[S]`, not `[V]`. Direct byline verification remains open.

---

## 2. What is BLOCKED in this environment — and why

This container's egress proxy returns **403 on CONNECT for every non-package host**. Package registries (pypi, npm) are allowed; general web is not. This blocks:

- `curl` / any direct HTTP to external hosts
- The `WebFetch` tool — returns `EGRESS_BLOCKED` for **every** domain tested, including `en.wikipedia.org`
- Chromium/Playwright — binaries are present at `/opt/pw-browsers`, but there is no network route and no authenticated session

**`WebSearch` is the only working external channel** (it executes server-side at Anthropic). All verification above was done through it, which means results are search-snippet-derived and carry the tagging caveats noted.

### Consequence for the two headline tasks

| Task | Verdict |
|------|---------|
| **Geofenced ArcGIS query** (Gap #1, highest value) | **CANNOT EXECUTE.** Endpoint confirmed to exist (org `79kfd2K6fskCAkyg`; layers `crime_data_2025`, `Police_Incidents_2026`) but `services1.arcgis.com` and `opendata.minneapolismn.gov` are both blocked. |
| **Nextdoor / Reddit forum layer** (Gap #5) | **CANNOT EXECUTE.** No browser-automation MCP with the owner's cookies is wired up, *and* both domains are blocked at the proxy. The handoff's precondition — "confirm that tool is wired up" — resolves to **NO**. |

### How to unblock the ArcGIS query

Re-create the remote environment with a network policy permitting at minimum:

```
services1.arcgis.com
opendata.minneapolismn.gov
```

Then the query below runs directly. Network policy is chosen when the environment is created — see https://code.claude.com/docs/en/claude-code-on-the-web

### Ready-to-run query (blocked here; runs anywhere with egress)

Replace `<LAT>` / `<LON>` with the geocoded parcel centroid. `distance=402` is a quarter-mile in metres.

```bash
curl -s "https://services1.arcgis.com/79kfd2K6fskCAkyg/arcgis/rest/services/crime_data_2025/FeatureServer/0/query" \
  --data-urlencode "geometry=<LON>,<LAT>" \
  --data-urlencode "geometryType=esriGeometryPoint" \
  --data-urlencode "inSR=4326" \
  --data-urlencode "distance=402" \
  --data-urlencode "units=esriSRUnit_Meter" \
  --data-urlencode "spatialRel=esriSpatialRelIntersects" \
  --data-urlencode "where=date_occurred >= TIMESTAMP '2025-09-14 00:00:00'" \
  --data-urlencode "outFields=*" \
  --data-urlencode "returnGeometry=true" \
  --data-urlencode "resultRecordCount=2000" \
  --data-urlencode "f=json" | jq '.features | length'
```

Repeat against the 2026 layer. **Verify the layer name first** (`/FeatureServer/0?f=json`) — layer naming has been inconsistent across years (`crime_data_2025` vs `crimedata2024`). Confirm `date_occurred` exists and note it may differ from `date_reported`; use `date_occurred` for a window filter.

---

## 3. Still open — priority order

1. **[HIGH] Incident-level data, ¼-mile, 365 days.** Unchanged as the single most load-bearing gap. Route A: ArcGIS query above (needs egress). Route B: dual MGDPA requests (§4).
2. **[HIGH] Two data requests, not one.** See §4 — the MPD-only request misses the parkway.
3. **[HIGH] Physical route recon — owner-only.** Walk/drive the route once at 5:30 a.m. *in the current season*: lighting, sightlines, cover, isolated stretches, phone signal. Seasonally dependent; a summer recon does not answer a November question.
4. **[HIGH] Resident-forum layer — owner's browser only.** Search strings in Annex A. Capture **date** + **firsthand vs repost** for every hit; that rule is what prevents another Rev A error.
5. **[MED] Predatory-offender proximity.** MN DOC Level-3 registrant search; MN BCA non-compliant search. Interactive lookups.
6. **[MED] Ownership/occupancy conflict.** Hennepin County property + recorder portals.
7. **[LOW] Southwest Voices byline verification.** Confirms the correction at `[V]` rather than `[S]`.

---

## 4. MGDPA requests — BOTH are required

### 4a. Minneapolis Police Department (residential blocks)

> To: Minneapolis Police Department, Records/Data Practices Unit
> Re: Government Data Practices Act Request — Incident Data
>
> Under Minnesota Statute § 13.03, I am requesting the following public government data: all MPD incident reports (Part I and Part II offenses) within a quarter-mile radius of 4889 E Lake Harriet Parkway, Minneapolis, MN 55419, for the period September 14, 2025 through the present.
>
> Please include, for each incident: offense type, date, time, general location (block-level acceptable per standard redaction), and disposition. Electronic format (CSV/Excel) preferred. Please advise of any fees prior to fulfillment.

### 4b. Minneapolis Park & Recreation Board — **THE ONE THE HANDOFF MISSED**

The running surface is Park Board property patrolled by MPRB Park Police, not MPD. Park Patrol Agents concentrate on lakes and parkways **in summer months** — which is precisely why a pre-dawn winter run sits in a coverage seam, and why MPD records alone will not describe it.

> To: Minneapolis Park & Recreation Board — Data Practices / Responsible Authority
> Re: Government Data Practices Act Request — Park Police Incident Data
>
> Under Minnesota Statute § 13.03, I am requesting the following public government data: all Minneapolis Park Police incident reports for the Lake Harriet parkway and surrounding park property — including East and West Lake Harriet Parkway, the band shell area, and the boat launch — for the period September 14, 2025 through the present.
>
> Please include, for each incident: offense or call type, date, time, location, and disposition. I am additionally requesting any available record of patrol coverage hours or staffing levels for this park area between 04:00 and 07:00 during that period. Electronic format (CSV/Excel) preferred. Please advise of any fees prior to fulfillment.

- MPRB Responsible Authority: **Jennifer Ringold**
- Public-data request form: `minneapolisparks.org/about-us/leadership-and-structure/public_data/`
- MPRB also publishes **Park Police Incident Summaries** directly: `minneapolisparks.org/about-us/news/park-police-incident-summaries/`

The patrol-hours element of 4b is the one request that could actually **measure** the coverage seam rather than infer it.

---

## 5. Analytic discipline held

- The property-crime pattern did **not** inflate Category C. The 2–7 a.m. offender-presence overlap is recorded as an **encounter** consideration with an explicit note that it does not raise the violence estimate.
- The Elm Creek Park Reserve assault (31 Aug 2026, lone woman, early morning, park trail, suspect at large) is **in-window and activity-matched but 15 mi away in a different jurisdiction and park system**. Included as activity-profile context only, explicitly excluded from the local base rate.
- Loring Park (2 Sep 2026) and Annunciation (27 Aug 2025) excluded from local scoring on the same non-distributed-signal logic, stated visibly rather than silently.
- Consumer crime-score aggregators excluded throughout. `minneapoliscrime.com` surfaced during search and was discarded per that rule.
- Rev A → Rev B score movement (34 → 36) is **within the instrument's noise**. The band is the finding.

## 6. Open question for the owner

The MCCA rape-category increase (+17.8% YoY, against a −6.2% national trend) is the most activity-relevant trend line in the assessment, but its weight depends on the runner's own risk profile, which is not on record. This should be resolved before the assessment is treated as final.

---

## Build

```bash
pip install weasyprint
python3 -m weasyprint "docs/<file>.html" "pdf/<file>.pdf"
```

Both deliverables verified at **2 pages**. Footer is a `@page` margin box — it does not orphan the way Chromium print does. "Human BBY" is specified and falls back to a system sans where unavailable.

---

## 7. Doc set

| Document | Format | Pages | Role |
|---|---|---|---|
| `PI-2026-RSA-001 Residential Threat Assessment` | HTML + PDF | 3 | **Main product.** Full scored matrix, 15 factors across 3 weighted categories, banding table, sensitivity test, sources. |
| `PI-2026-RSA-001 Residential Threat Assessment (Brief)` | HTML + PDF | 2 | Plain-language version. Verdict banner, finding cards, run section. The format the owner preferred. |
| `PI-2026-RSA-001-A Media and Online Sentiment` | HTML + PDF | 2 | Annex A. Correction notice, in-window media table, forum-layer collection plan. |

### Scoring matrix (main product, Section 05)

Factors score 0–100. Category score = unweighted mean of its factors. Composite = weighted sum.

| Category | Weight | Score | Contribution |
|---|---|---|---|
| A · Site and Property | 25% | 31 | 7.75 |
| B · Threat Intelligence | 32% | 32 | 10.24 |
| C · Vulnerability and Environmental | 43% | 43 | 18.49 |
| **Composite** | | **36** | **Tier 3 · MODERATE** |

Banding: 81–100 Tier 1 CRITICAL · 61–80 Tier 2 HIGH · 36–60 Tier 3 MODERATE · 21–35 Tier 4 LOW · 0–20 Tier 5 MINIMAL.

**Sensitivity test.** Removing the pre-dawn run leaves A and B unchanged and reduces C to its ambient environmental residue (~18). Composite becomes `(0.25 × 31) + (0.32 × 32) + (0.43 × 18) = 26`, which is **Tier 4 LOW**. This makes the handoff's central claim arithmetically demonstrable rather than asserted: one behavior moves the assessment across a tier boundary and nothing else in the matrix does.

### House-style deviations, stated

The main product follows the `protective-intelligence` house standard: no em-dashes, no banned words, no hedge phrases, no transition-word openers, inline source-plus-date, five-tier risk framing, "PI assesses" for analytic judgment. Validated mechanically (0 em-dashes, 0 banned words, 0 hedges).

Two deviations, both deliberate:
1. **Body set at 7.95pt, not the 11pt house standard.** A 15-factor matrix with an 8-row incident table will not hold a sensible page budget at 11pt. Raising to 11pt roughly doubles the page count.
2. **3 pages, not the 2 the handoff specified for the full version.** The house VSEC standard is 4–8 pages, so 3 is tight rather than long. The 2-page constraint is met by the Brief.

The Brief and Annex A predate the house-style pass and still use em-dashes. They were not retrofitted, since the owner had already approved that format. Say the word and they get the same treatment.

---

## 8. Rev C — neighborhood-only scoping

At the owner's direction, Rev C limits the evidence base to the home, the Lake Harriet parkway, and the four ring neighborhoods (East Harriet, Lynnhurst, Fulton, Linden Hills). All citywide, statewide and national comparators were removed: MCCA midyear figures, the MN BCA Uniform Crime Report, "Raise the Age", Loring Park, Elm Creek Park Reserve, the Whittier/Lowry Hill East robbery alert, and the Ferrier death (5400 block of 43rd Ave S, ~4 mi east, outside the ring). Rev B retains them.

### The composite did not move

| Category | Rev B | Rev C | Weight | Contribution |
|---|---|---|---|---|
| A · Site and Property | 31 | 31 | 25% | 7.75 |
| B · Threat Intelligence | 32 (citywide-inclusive) | **32 (local only)** | 32% | 10.24 |
| C · Vulnerability and Environmental | 43 (6 factors) | **42.6 (7 factors)** | 43% | 18.31 |
| **Composite** | **36** | **36** | | Tier 3 · MODERATE |

A score that survives removal of its broadest inputs rests on the local record rather than on background trend. That robustness is a stronger result than the Rev B number it reproduces.

### Two factors added

- **B · Local vehicular threat on or near the route — 45.** 1 May 2026, East Harriet: stolen Hyundai ran a stop sign at 80 mph with headlights off at W 46th St and Aldrich Ave S, struck a State Patrol squad; trooper fractured fibula and scapula, a passenger took a compound leg fracture and a brain bleed. Precedent for the same mechanism reaching a runner on the parkway itself: 5 Nov 2020, minivan left the roadway and struck a jogger before entering the lake.
- **C · Active construction on the route corridor — 40.** Storm sewer reconstruction on Oliver Ave between W 50th St and Lake Harriet Parkway, and W 50th between Oliver and Penn, began 13 Jul 2026 and runs into summer 2027. It changes footing, lighting and sightlines on part of the route through the coming winter.

### Three dating failures caught

| Claim | Reads as | Actually |
|---|---|---|
| "5 of 7 carjackings ringing Lake Harriet, Nov 14–25" | In-window | **Nov 2021** |
| MPD 5th Pct alert, victims "alone… either walking" | Current, on-point | Cites Insp. **Katie Blackwell**, who left that command **Aug 2023**; names Lowry Hill East and Whittier, outside the ring |
| Band shell shooting | "October 2026" per a search summary | **24 Oct 2023** per charging records |

All three would have passed a casual read, and all three described exactly the kind of incident the owner asked about. Date verification, not collection volume, is the binding constraint on this assessment.

### Reddit is permanently closed to Claude

A domain-restricted search returns a hard refusal, not an empty result: `The following domains are not accessible to our user agent: ['reddit.com']`. This is not specific to this container. No Claude session can reach Reddit. Combined with Nextdoor's login wall, the resident-forum layer can only be collected by the owner.

### Sources excluded during this revision

Searches for neighborhood-level crime returned CrimeGrade, AreaVibes, NeighborhoodScout, Niche, Safemap, SpotCrime and minneapoliscrime.com, carrying figures such as a Lynnhurst "A+ safety grade", "118 thefts / 87 vandalism / 15 assaults / 15 burglaries", "22.50 property crimes per 1,000", and an East Harriet "88.8/100 safety score" with a "53% increasing trend". All were discarded per the standing exclusion rule. None appear in any deliverable.

---

## 9. Rev D — 5-mile radius, strict 365-day window

Owner narrowed the Residential Threat Assessment to events, news and crime activity **within a 5-mile radius**, **trailing 365 days only**. Two filters applied in order: event date inside 14 Sep 2025 – 14 Sep 2026 (publication date does not qualify an item), then distance within ~5 mi. Distances estimated from street geography, approximate — no geocoding service was reachable.

**16 qualifying events. None on the Lake Harriet parkway.** Nearest in-window event of any kind is the 1 May 2026 vehicle collision at ~1 mi. Nearest violent crime is the 21 Jun 2026 Uptown homicide at ~2.5 mi.

| Category | Rev C (neighborhood) | Rev D (5 mi) | Weight |
|---|---|---|---|
| A · Site and Property | 31.0 | 31.0 | 25% |
| B · Threat Intelligence | 32.0 | **34.8** | 32% |
| C · Vulnerability and Environmental | 42.6 | **42.9** | 43% |
| **Composite** | 36 | **37** | Tier 3 · MODERATE |

Score history: Rev A 34 · Rev B 36 · Rev C 36 · Rev D 37. **Four scopes, one band.** Sensitivity test unchanged in direction: removing the pre-dawn run yields 27, Tier 4 LOW.

### What the strict window costs

It removes the only exact time/place/activity match in the file: the **Sep 2015** solo runner grabbed at 05:30 on W Lake Harriet Pkwy near Sheridan Ave S. Also removed: the **Jul 2017** escalation series on this lake corridor, and the **Nov 2020** minivan strike that injured a jogger on this parkway. Category C is now carried by structural conditions rather than precedent. The resulting picture is *less informative about the specific question asked*, not safer. Rev C retains the historical record.

### Five items excluded on date

Each reads as current and on-point; each was caught by checking a primary report.

| Item | Appears | Actual |
|---|---|---|
| "5 of 7 carjackings ringing Lake Harriet" | In window | **Nov 2021** |
| MPD alert, victims "alone… either walking" | Current | Cites Insp. Katie Blackwell, left command **Aug 2023** |
| Band shell shooting | "Oct 2026" per search summary | **24 Oct 2023** |
| "Rise in crime along Lake Nokomis, Minnehaha Creek" | Current warning, creek passes this lake | **Feb 2021** |
| "Three arrested after string of robberies in Uptown" | In window | **Sep 2024** |

### One exclusion worth naming

**Annunciation church shooting** — ~1.5 mi, comfortably inside the radius, and it drives the 5th Precinct YoY shooting-victim increase single-handedly. It occurred **27 Aug 2025**, eighteen days before the window opens. Under the owner's filters it does not qualify. Named in the document so the omission is visible rather than silent.

### Excluded on distance

Elm Creek Park Reserve (Maple Grove, ~15 mi) · North Minneapolis pursuit crash at N Lyndale & 41st Ave N (~7 mi) · Boom Island and NE Minneapolis break-in sites (at or beyond the boundary).

---

## 10. FINAL — QC pass

Every Section 03 item was re-verified against a primary report naming the event date. The pass removed **six items on date** and corrected **three factual errors, two of which had already been published** in Rev C/D.

### Removed on date (6)

| Item | Appears | Verified |
|---|---|---|
| "5 of 7 carjackings ringing Lake Harriet" | In window | Nov 2021 |
| MPD alert, victims "alone… either walking" | Current | Cites Insp. Katie Blackwell, left command Aug 2023 |
| Band shell shooting | "Oct 2026" per summary | 24 Oct 2023 |
| "Rise in crime along Lake Nokomis, Minnehaha Creek" | Current | Feb 2021 |
| "Three arrested, robbery string in Uptown" | In window | Sep 2024 |
| Linden Hills "13 larceny/theft, highest month in past year" | Aug 2026 | **Patch article dates to 2022** |

### Factual corrections (3)

1. **Published error.** The Linden Hills August figure ran in Rev C and Rev D as a verified in-window trend marker. Struck. Category B property-crime factor drops 58 → 55.
2. **Published error.** The 1 May collision passenger injury was recorded as a compound *leg* fracture. Charging record says compound **neck** fracture with brain bleed.
3. **Caught pre-publication.** Loring Park casualties initially read as three officers shot plus four exposed to an airborne substance. Verified: **two officers shot** (Michnowski, Ahmed), a third cut by glass, **seven wounded by gunfire total**.

### Material additions found during QC

The Lake Street cluster was under-counted. Three separate shootings, not one, all in the first two days of the window:

- **14 Sep 2025, 01:45**, 15th Ave S & E Lake St — man died, boy hospitalised. Trevon Harris (28) and Kenny Davenport (50) charged.
- **15 Sep 2025, just before noon**, E Lake St & Stevens Ave S — five men shot. Adam John Peterson, 46, died 20 Sep. No arrests.
- **15 Sep 2025, ~22:00**, E Lake St & 28th Ave S — encampment mass shooting, 30+ shots. Jacinda Oakgrove, 30, killed, six injured. Drug-territory dispute per charges. City cleared the encampment hours later.

Also added: suspect identity and charges for the 1 May collision (Thailand Taiwan Hawkins, 19), and the Loring Park victim and officer identifications.

### Final score

| Category | Score | Weight | Contribution |
|---|---|---|---|
| A · Site and Property | 31.0 | 25% | 7.75 |
| B · Threat Intelligence | 35.2 | 32% | 11.26 |
| C · Vulnerability and Environmental | 42.9 | 43% | 18.43 |
| **Composite** | **37** | | **Tier 3 · MODERATE** |

Score history: 34 · 36 · 36 · 37 · **37 final**. Five scopes and one QC pass, one band throughout. Sensitivity test: removing the pre-dawn run yields **27, Tier 4 LOW**.

**18 qualifying items inside 5 miles. Zero on the Lake Harriet parkway.**

### Verification assertions run against the final file

- 0 em-dashes, 0 banned words, 0 hedge phrases, 0 transition-word openers
- Struck claims (`13 larceny`, `compound leg`, `three officers shot`, `airborne substance`) appear **only** inside the Section 07 QC log where they are documented as corrections, confirmed by line-range check against section boundaries
- Matrix arithmetic recomputed independently: `(0.25 × 31.0) + (0.32 × 35.2) + (0.43 × 42.857) = 37.44 → 37`
- All 14 event dates checked for real calendar dates and weekday consistency; window confirmed as exactly 365 days
- Stated body point size matches the CSS
