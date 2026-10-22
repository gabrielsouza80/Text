# QA Validation Report | Property Page — Primary Highlights

**Work items:** SKIP-2171 (Property Page > Primary Highlights), SKIP-2301 (Luca > Property Page Highlights)  
**Related test work item:** SKIP-2248 (Test — Property Page Primary Highlights)  
**Environment:** Iglu Ski Staging; `skip-2301.staging.igluski.com` observed in the browser  
**Feature variant requested for testing:** `primary-highlights=B`  
**Report compiled:** 22 October 2026  
**Scope:** Manual UI verification and read-only SQL Server checks of the `Ski` database  
**Overall QA recommendation:** **IN TEST — sign-off pending the three priority findings listed in Section 0**
**Requirements traceability:** Every outstanding finding is cross-referenced to wording visible in the Jira ticket screenshots (Section 1A).

> **Evidence standard:** This report distinguishes what was seen on screen or returned by SQL from interpretation, requirements questions and untested cases. It does not establish a root cause. The A/B cookie was requested for testing, but its exact value was not independently captured in every screenshot. Screenshots were supplied during the QA session; they are **not embedded in this Markdown file**. Holiday IDs below were read from visible offer pages or browser URLs, not inferred from SQL joins.

---

## 0. ACTION REQUIRED — PRIORITY FINDINGS (READ THIS FIRST)

**QA status: IN TEST — NOT READY FOR PASS.** Five offers were reviewed. The feature works in several positive examples, but the three cases below need **product clarification and/or a targeted fix** before final sign-off. **No root cause has been verified.**

### PRIORITY 1 — TRANSFERS HIGHLIGHT MISSING | Club Med Les Arcs Panorama

**Finding F-01 · Most direct mismatch with a written rule · Property `36707` · Holiday `203290784` · 2/4 cards**

> **Ticket requirement — SKIP-2171, Transfers:** “IF No transfer is included THEN show ‘Not included’” and “show shortest transfer time for the resort”. Also relates to **REQ-01** (four primary highlights).

- **Expected if the rule applies to this offer:** a **Transfers** card labelled **“Not included”**, with the shortest available resort transfer duration.
- **Observed:** the booking summary states **“Transfers & Flights: Not included”** (accommodation-only offer), but **no Transfers card or alternative duration** is shown.
- **User-facing consequence:** the planned transfer information is **not available in the highlights area**; the offer summary still discloses that transfers are not included.
- **Decision / action needed:** **PO:** confirm if the rule applies to accommodation-only offers. **Development:** if applicable, check whether an alternative resort transfer duration exists and whether the card should render; then correct and rerun this specific offer.
- **Evidence:** UI offer summary and highlights screenshots; see **REQ-05, F-01, SC-12, Section 5.3**. No claim is made about whether the cause lies in data, logic or deployment.

### PRIORITY 2 — RESORT LOCATION FALLBACK / CARD PLACEMENT | Rodella Hotel

**Finding F-02 · Requirement placement is ambiguous · Property `2549` · Holiday `158518187` · 3/4 cards**

> **Ticket requirement — SKIP-2171, Resort Centre Location:** “IF the walk is over 1000m THEN distance to bus stop is shown instead”. Also relates to **REQ-01**.

- **Expected subject to PO design confirmation:** when the centre is more than 1,000 m away, show the bus-stop distance **in the intended resort-location experience** instead of the standard walking-time message.
- **Observed:** SQL records `TownDistance=1200`, `TownDistanceMinutes=12` and `SkiBusStopDistance=10`. **“Ski Bus: 10m away”** appears under **Ski Access**, but **Resort Location is absent**.
- **User-facing consequence:** the bus-stop value **is displayed**, although not in the expected location card; this is **not yet a confirmed separate bug**.
- **Decision / action needed:** **PO/design:** specify whether Ski Access already fulfils the fallback or whether Resort Location must still appear. **Development:** adjust only if the confirmed specification requires it.
- **Evidence:** SQL and highlights screenshots; see **REQ-06, F-02, SC-08, Section 5.5**. Do not confuse this with the separate *slopes-to-bus-stop* rule (**REQ-07**).

### PRIORITY 3 — ONLY ONE CARD / NULL FALLBACK | Apartment Petit Pont

**Finding F-03 · Missing-data rule not defined · Property `33586` · Holiday `127580406` · 1/4 cards**

> **Ticket requirement — SKIP-2301, Acceptance Criteria:** “4 primary highlights for property page displayed”. Related to **SKIP-2171**'s over-1,000 m centre fallback (**REQ-06**).

- **Expected from the design wording:** four highlights, **unless exceptions for unavailable or inapplicable inputs are approved**.
- **Observed:** only **Snow Security** appears. SQL records `TownDistance=1100`, `TownDistanceMinutes=11`, and `SkiBusStopDistance=NULL`.
- **User-facing consequence:** substantially less information is available in the highlights row; the source data does not currently establish a valid bus-stop fallback.
- **Decision / action needed:** **PO:** decide what to display when a field is `NULL`, and whether four cards are mandatory. **Development:** apply the agreed fallback or intentional-hide behaviour and rerun the offer.
- **Evidence:** SQL and UI screenshots; see **REQ-01, REQ-06, F-03, SC-09, Section 5.4**. **`NULL` is missing data, not zero metres.**

### Shared acceptance decision — four cards or conditional cards?

**SKIP-2301 says “4 primary highlights for property page displayed”.** The observed counts were **4/4** (Villa Cary), **4/4** (Hotel Sportiv), **3/4** (Rodella), **2/4** (Club Med), and **1/4** (Petit Pont). This difference is **evidence of an acceptance-criteria question**, not automatically three independent implementation defects. **PO decision required:** are all four cards mandatory for every property, or should missing/inapplicable cards be omitted?

**Recommended next step:** Obtain the PO's answers to the three questions above, confirm relevant data availability with Development, and rerun **Club Med → Rodella → Petit Pont**. Do not change status to PASS on the strength of other successful examples alone.

---

## Quick navigation

- **[0. Action required — priority findings](#0-action-required--priority-findings-read-this-first)** — one-page review for PO and developers
- [1. Executive summary](#1-executive-summary-for-the-product-owner-and-developers) — prioritised outstanding issues
- [1A. Requirements-to-findings traceability](#1a-requirements-to-findings-traceability-exact-jira-wording) — exact ticket wording for each observation
- [2. Reproduction guide](#2-reproduction-guide) — how to repeat the tests
- [3. Property and Holiday ID register](#3-property-and-holiday-id-register) — the five inspected offers
- [4. Acceptance criteria and scenario matrix](#4-acceptance-criteria-and-scenario-matrix) — PASS, unresolved and untested conditions
- [5. Key evidence and technical observations](#5-key-evidence-and-technical-observations) — property-by-property findings
- [6. Read-only SQL reproduction pack](#6-read-only-sql-reproduction-pack) — SQL queries Q01–Q13
- [7. Questions for Product / Development](#7-questions-for-product--development) — decisions needed
- [8. Remaining coverage / next test actions](#8-remaining-coverage--next-test-actions)
- [9. Suggested concise Jira comment](#9-suggested-concise-jira-comment-uk-english)

## 1. Executive summary (for the Product Owner and developers)

**Start with [Section 0](#0-action-required--priority-findings-read-this-first): three outstanding findings, each mapped to its exact Jira clause and a specific decision or retest.**

The Primary Highlights component was observed below the image gallery on multiple Staging property pages. Positive examples were confirmed for snow information, ski-in/ski-out or ski-bus access, included transfers and the available resort-location values. **Three properties did not display all four highlights**, and one specific **“Transfers not included”** scenario could not display the content described in the requirements.

### Findings requiring action or clarification

| Priority | Finding | Reproduction reference | Observed outcome | Why it matters |
| --- | --- | --- | --- | --- |
| **Highest priority — requirement mismatch to investigate** | Transfer highlight absent when transfers are not included | **Club Med Les Arcs Panorama** — Property `36707`; Holiday `203290784` | Offer panel says **“Transfers & Flights: Not included”**, but no **Transfers** highlight or fallback transfer time is shown; only 2 highlights appear | The SKIP-2171 specification describes a **“Not included”** state with an alternative transfer time. Confirm whether this applies to accommodation-only bookings and whether the resort's transfer data is available. |
| **Requirements clarification** | Resort Location absent with town distance above 1,000 m and valid bus-stop data | **Rodella Hotel** — Property `2549`; Holiday `158518187` | SQL: 1,200 m / 12 min to town and 10 m to ski-bus stop. UI: **“Ski Bus: 10m away”** under *Ski Access*, but no separate *Resort Location* highlight; 3 highlights appear | Clarify whether the >1,000 m fallback belongs in *Resort Location*, whether *Ski Access* replaces it, or whether omission is intentional. |
| **Missing-data clarification** | Only one highlight when fallback bus-stop information is absent | **Apartment Petit Pont** — Property `33586`; Holiday `127580406` | SQL: 1,100 m / 11 min to town; `SkiBusStopDistance=NULL`. UI: only *Snow Security* is visible | Behaviour is **inconclusive** without a documented rule for missing fallback data. |
| **Cross-cutting** | Number of highlights differs across properties | Villa Cary `4/4`; Hotel Sportiv `4/4`; Rodella `3/4`; Club Med `2/4`; Petit Pont `1/4` | Not all sampled pages display four cards | SKIP-2301 explicitly mentions **four** highlights. PO/design confirmation is needed on whether cards may be suppressed when data or applicable services are missing. |

**Disposition:** Do **not** mark SKIP-2171 / SKIP-2301 as fully passed. The missing Transfers highlight is the strongest observed requirement mismatch. The other absences should be reviewed against intended conditional behaviour before being classified as independent defects.

## 1A. Requirements-to-findings traceability (exact Jira wording)

**Source:** The following quotations were transcribed from the screenshots of the **SKIP-2171** and **SKIP-2301** Jira descriptions / acceptance criteria provided during QA. Only the relevant sentence or clause is quoted. These are **requirement references**, not claims about the implementation, the cause of an issue, or product decisions that have not been made.

| Requirement ID | Source | Exact relevant wording shown in Jira | Meaning for QA |
| --- | --- | --- | --- |
| **REQ-01** | **SKIP-2301 — Acceptance Criteria** | “4 primary highlights for property page displayed” | The specified design targets four displayed highlights. Whether unavailable data justifies fewer requires PO confirmation. |
| **REQ-02** | **SKIP-2171 — Acceptance Criteria** | “Primary Highlights should be shown on the property page” | The component should be visible on the property page. |
| **REQ-03** | **SKIP-2171 — Acceptance Criteria** | “These should be shown underneath the image gallery” | Position beneath the gallery was verified on the sampled pages. |
| **REQ-04** | **SKIP-2171 — Acceptance Criteria** | “The data in each of the boxes should be a reflection of the data described in the description” | Contents need to match the relevant rules and input data; it does not define behaviour when inputs are missing. |
| **REQ-05** | **SKIP-2171 — Transfers description** | “IF No transfer is included THEN show ‘Not included’” and “show shortest transfer time for the resort” | A not-included state and alternative duration are described. Applicability to accommodation-only offers needs clarification. |
| **REQ-06** | **SKIP-2171 — Resort Centre Location description** | “IF the walk is over 1000m THEN distance to bus stop is shown instead” | A bus-stop fallback is specified for centre-distance cases exceeding 1,000 m; the precise placement / omission behaviour needs confirmation. |
| **REQ-07** | **SKIP-2171 — Ski Lift Location description** | “IF the distance is over 1000m from the slopes THEN show distance to bus stop instead” | Separate ski-access fallback. **Do not confuse this with REQ-06**: slopes and resort centre are distinct conditions. |
| **REQ-08** | **SKIP-2171 — Resort Centre Location description** | “IF Under 6 minutes walk it should then show ‘You're in the centre’” | The example with 2 minutes displays a different, similar phrase. Record as a wording observation, not automatically a blocking defect. |

### Finding F-01 — Club Med Les Arcs Panorama (strongest mismatch)

- **Relevant ticket wording:** **REQ-05** and **REQ-01** (plus the general data-consistency rule **REQ-04**).
- **Observed evidence:** Property `36707`; Holiday `203290784`; accommodation-only offer states **“Transfers & Flights: Not included”**. The Primary Highlights row shows **2 of 4** cards, with no Transfers card and no alternative transfer duration.
- **Expected from the written requirement, if applicable:** A **Transfers** highlight stating **“Not included”** and the shortest resort transfer time.
- **Assessment:** **Observed mismatch with the written Transfers rule; scope/data clarification required.** Confirm whether the rule includes accommodation-only bookings and whether a resort-level fallback transfer time is available. No root cause is established.

### Finding F-02 — Rodella Hotel (location/fallback ambiguity)

- **Relevant ticket wording:** **REQ-06** for **Resort Centre Location**, **REQ-01** for four cards. **REQ-07** applies to ski access, not automatically to resort centre.
- **Observed evidence:** Property `2549`; Holiday `158518187`; SQL shows `TownDistance=1200`, `TownDistanceMinutes=12`, `SkiBusStopDistance=10`. Site shows **“Ski Bus: 10m away”** in **Ski Access**; **Resort Location** is absent; **3 of 4** highlights appear.
- **Expected from the written requirement, subject to placement confirmation:** Over 1,000 m from the **resort centre**, show bus-stop distance instead of centre walking distance.
- **Assessment:** **Clarification required, not a confirmed independent bug.** The 10 m bus-stop value is visible in Ski Access. Ask whether this satisfies the intended fallback or Resort Location must remain visible.

### Finding F-03 — Apartment Petit Pont (missing-data behaviour)

- **Relevant ticket wording:** **REQ-01** for four cards, and **REQ-06** for the centre-distance fallback.
- **Observed evidence:** Property `33586`; Holiday `127580406`; SQL shows `TownDistance=1100`, `TownDistanceMinutes=11`, `SkiBusStopDistance=NULL`. Only **1 of 4** highlights is visible.
- **Expected from the written requirement:** Four highlights are specified in the design; the centre-distance fallback is described, **but no rule for `NULL` bus-stop data is given in the supplied Jira description**.
- **Assessment:** **Unresolved requirements question, not a proven application defect.** Confirm whether cards should be hidden or display alternate/fallback copy when input data is missing.

### Cross-cutting observation — Four-card count

- **Relevant ticket wording:** **REQ-01**.
- **Observed counts:** Villa Cary **4/4**, Hotel Sportiv **4/4**, Rodella **3/4**, Club Med **2/4**, Apartment Petit Pont **1/4**.
- **Assessment:** The observed count differs from the explicit SKIP-2301 acceptance criterion in three examples. However, SKIP-2171 includes conditional content and missing inputs; ask the PO whether four cards are **mandatory** or **a design target subject to valid/applicable data** before marking all three examples as defects.

### Additional minor copy observation — Villa Cary

- **Relevant ticket wording:** **REQ-08**.
- **Observed evidence:** SQL shows **2 minutes** to the centre; the UI displays **“In Resort Centre”** rather than the specified **“You're in the centre”**.
- **Assessment:** Possible wording discrepancy. The concept is broadly equivalent, so verify whether exact copy is required; do not conflate this with the three outstanding findings above.

## 2. Reproduction guide

1. Open the applicable property offer in the `SKIP-2301` Staging branch and use the test variant `primary-highlights=B` as requested by the development team. Check the actual cookie in browser DevTools if the result is uncertain.
2. Scroll to **Primary Highlights**, immediately beneath the image gallery.
3. Record the **number of cards**, each card's **label and value**, and its **hover tooltip**.
4. Compare *Transfers* against the offer's **What's Included** panel. In particular, distinguish package transfers from an accommodation-only offer.
5. Use the read-only SQL checks in Section 6 to compare the displayed resort/property data with `dbo.Property`, `dbo.Resort` and the applicable feature associations.
6. Capture the property URL including its Holiday ID, the complete highlights row, relevant tooltips, the booking summary, and the SQL result where required. Do not treat an unavailable highlight as proof of the underlying reason.

**Known branch URL captured:**  
`https://skip-2301.staging.igluski.com/ski-resorts/italy/selva-val-gardena/rodella-hotel_2549?holidayId=158518187`

The other Holiday IDs were captured from visible page URLs or offer panels, but this report does not invent their full branch-specific URLs.

## 3. Property and Holiday ID register

These five properties were opened for functional inspection during the session. **“Validated” means the recorded UI scenario was examined, not that the property or complete work item passed every acceptance criterion.**

| Property | Property ID | Holiday ID captured | Context / verified highlights | Observed count |
| --- | ---: | ---: | --- | ---: |
| **Villa Cary Hotel** — Sauze d'Oulx, Italy | `2541` | `2025028098` | Snow Security; Ski Access; Resort Location (**“In Resort Centre”**); Transfers (**included**) | **4/4** |
| **Hotel Sportiv by VAYA** — Obergurgl, Austria | `1386` | `166713775` | Snow Sure Skiing; Ski Access (**“Ski-in / Ski-out”**); Resort Location (**“6 mins walk to centre”**); Transfers (**included**) | **4/4** |
| **Club Med Les Arcs Panorama** — Les Arcs 1600, France | `36707` | `203290784` | Snow Sure Skiing; Ski Access (**“Ski-in / Ski-out”**). **Resort Location and Transfers absent**; offer says **“Transfers & Flights: Not included”** | **2/4** |
| **Apartment Petit Pont** — Villars, Switzerland | `33586` | `127580406` | Snow Security only. **Ski Access, Resort Location and Transfers absent**; accommodation-only offer | **1/4** |
| **Rodella Hotel** — Selva Val Gardena, Italy | `2549` | `158518187` | Snow Sure Skiing; Ski Access (**“Ski Bus: 10m away”**); Transfers (**included**). **Resort Location absent** | **3/4** |

**SQL-only candidate, not functionally validated:** Bellevarde Lodge - Cognac (`PropertyId=15174`, `TownDistance=1100`, `TownDistanceMinutes=11`, `SkiBusStopDistance=50`). No Holiday ID or page-level result was established for this property.

## 4. Acceptance criteria and scenario matrix

**Requirement source:** SKIP-2171 conditional highlight descriptions and SKIP-2301 acceptance criteria. **Each outstanding finding is traced to exact wording in Section 1A (REQ-01–REQ-08).** The matrix below reports only evidence collected during the QA session.

| ID | Scenario / stated expectation | Evidence | Outcome |
| --- | --- | --- | --- |
| **SC-01** | Highlights are placed beneath the image gallery | Location confirmed on several sampled pages | **PASS — observed pages** |
| **SC-02** | **REQ-01:** Four highlights displayed | Villa Cary and Hotel Sportiv: 4; Rodella: 3; Club Med: 2; Petit Pont: 1 | **NOT MET on three samples; conditional rules need clarification** |
| **SC-03** | Snow Sure information reflects resort height / highest lift / snow cannons where applicable | Villa Cary: altitude `1503`, maximum lift `2840`, snow cannons enabled. Club Med: `1600` / `3250` / enabled. Snow tooltips observed. Rodella and Hotel Sportiv snow highlights also visible | **PASS — positive examples only** |
| **SC-04** | Ski-in/Ski-out property displays direct slope access | Club Med and Hotel Sportiv display **“Ski-in / Ski-out”**; Club Med's corresponding feature is present in SQL | **PASS — combined condition only** |
| **SC-05** | Where relevant, show ski-bus access and distance | Rodella displays **“Ski Bus: 10m away”** and tooltip describing a nearby local shuttle stop; SQL confirms 10 m | **PASS — sampled ski-bus condition** |
| **SC-06** | **REQ-08:** Resort location for short town-centre distance; under 6 minutes has specific centre wording in specification | Villa Cary: SQL `200m / 2min`; UI says **“In Resort Centre”**, not the exact specified phrase **“You're in the centre”** | **FUNCTIONALLY PRESENT — wording not an exact match** |
| **SC-07** | At 6 minutes (not *under* 6), display walking time | Hotel Sportiv: SQL `600m / 6min`; UI **“6 mins walk to centre”** | **PASS — boundary example** |
| **SC-08** | **REQ-06 (town centre, distinct from REQ-07 for ski access):** Above 1,000 m, show bus-stop information instead of normal town-centre walking information | Rodella: SQL `1200m / 12min`, bus stop `10m`; 10 m appears under *Ski Access*; *Resort Location* absent | **UNRESOLVED — confirm intended card/fallback** |
| **SC-09** | **REQ-01 / REQ-06 (NULL behaviour unspecified):** Above 1,000 m with no bus-stop distance | Petit Pont: `1100m / 11min`; bus-stop distance `NULL`; *Resort Location* absent | **INCONCLUSIVE — missing-data rule not specified** |
| **SC-10** | Location card when town distance itself is missing | Club Med: `TownDistance=NULL`, `TownDistanceMinutes=NULL`; *Resort Location* absent | **INCONCLUSIVE — insufficient input data** |
| **SC-11** | Included transfers show an inclusion state and duration | Villa Cary, Hotel Sportiv and Rodella show inclusion. Rodella tooltip: shared airport transfer (~1h 45m) | **PASS — sampled included offers** |
| **SC-12** | **REQ-05:** Not-included transfers show a **“Not included”** state and fallback/shortest transfer time | Club Med's offer panel explicitly says not included; **Transfers card not present** | **NOT MET in observed UI — applicability/data to confirm** |
| **SC-13** | All distinct conditional alternatives | Non-Snow-Sure resort; ski-in only; ski-out only; missing data/fallback combinations; transfer time when not included | **NOT FULLY TESTED** |

**Important nuance:** SC-12 is a UI-to-requirement mismatch, not proof of an application code defect or a specific backend failure. The accommodation-only context may require an explicit product decision.

## 5. Key evidence and technical observations

### 5.1 Villa Cary Hotel — `PropertyId=2541`, `HolidayId=2025028098`

- Database: `ResortId=3484`, `Altitude=1503`, `MaxLift=2840`, `HasSnowCannons=1`, `TownDistance=200`, `TownDistanceMinutes=2`.
- Four highlights were displayed. The location highlight reads **“In Resort Centre”**.
- Snow and transfer tooltips were observed. The interface's resort-location message matches the intent of a central location, **but not the exact under-six-minute wording in the specification**.

### 5.2 Hotel Sportiv by VAYA — `PropertyId=1386`, `HolidayId=166713775`

- SQL search found `TownDistance=600` and `TownDistanceMinutes=6`.
- Four highlights displayed, including **“Ski-in / Ski-out”**, **“6 mins walk to centre”**, and transfers included.
- Hover tooltips for the visible highlights were inspected.

### 5.3 Club Med Les Arcs Panorama — `PropertyId=36707`, `HolidayId=203290784`

- Database: `ResortId=25622`, `Altitude=1600`, `MaxLift=3250`, `HasSnowCannons=1`; `TownDistance` and `TownDistanceMinutes` both `NULL`.
- `dbo.Feature` / `dbo.FeatureType` confirms **FeatureTypeId 135 — Ski in/Ski out**; **FeatureTypeId 88 — Snow sure** was found at the associated resort.
- UI: exactly two visible highlights (**Snow Sure Skiing** and **Ski Access**).
- Offer: accommodation only; **“Transfers & Flights: Not included”**.
- **Missing:** Transfers highlight, “Not included” highlight text, alternative transfer duration; Resort Location highlight also absent. The location omission is not independently classified as a defect because the town-distance values are missing.

### 5.4 Apartment Petit Pont — `PropertyId=33586`, `HolidayId=127580406`

- Database: `TownDistance=1100`, `TownDistanceMinutes=11`, `SkiBusStopDistance=NULL`.
- UI: only **Snow Security** visible; offer is accommodation only.
- Further requirements clarification is needed before determining whether a bus-stop fallback must appear despite missing bus-stop data. The absent Transfers highlight is also not independently diagnosed from this example.

### 5.5 Rodella Hotel — `PropertyId=2549`, `HolidayId=158518187`

- Database: `TownDistance=1200`, `TownDistanceMinutes=12`, `SkiBusStopDistance=10`.
- UI: three highlights — **Snow Sure Skiing**, **Ski Access: Ski Bus: 10m away**, **Transfers: Included in package**.
- Ski Access tooltip states that a local resort shuttle bus stop is nearby. Transfer tooltip describes a shared airport transfer of approximately **1h 45m**.
- **Missing:** Resort Location. The bus-stop information is displayed, but under **Ski Access**, not Resort Location. Requires design/PO confirmation rather than an assumed root cause.
- An earlier attempt produced a blank page on the branch; subsequent screenshots show the same branch rendering successfully. **Do not carry forward the blank page as a confirmed defect.**

## 6. Read-only SQL reproduction pack

All commands below are `SELECT` queries and can be executed in SQL Server Management Studio against **`Ski` on the authorised Staging database**. They do not update data or trigger an exporter. Check the connected server/environment before running. Statements consolidate SQL shown or used during the session; some discovery queries were normalised for readability.

### Q01 — Identify relevant Property and Resort columns

```sql
USE [Ski];

SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME IN ('Property', 'Resort')
  AND (
       COLUMN_NAME LIKE '%Lift%'
    OR COLUMN_NAME LIKE '%Minut%'
    OR COLUMN_NAME LIKE '%Height%'
    OR COLUMN_NAME LIKE '%Altitude%'
    OR COLUMN_NAME LIKE '%Snow%'
  )
ORDER BY TABLE_NAME, COLUMN_NAME;
```

### Q02 — Verify property-to-resort data and Snow Sure inputs

```sql
USE [Ski];

SELECT
    p.PropertyId,
    p.Name AS PropertyName,
    r.ResortId,
    r.Name AS ResortName,
    r.Altitude,
    r.MaxLift,
    r.HasSnowCannons
FROM dbo.Property AS p
INNER JOIN dbo.Resort AS r
    ON p.ResortId = r.ResortId
WHERE p.PropertyId IN (2541, 36707);
```

**Captured results:** Villa Cary `2541` / Resort `3484` / altitude `1503` / max lift `2840` / snow cannons `1`; Club Med `36707` / Resort `25622` / altitude `1600` / max lift `3250` / snow cannons `1`.

### Q03 — Compare centre distance, walking time and bus-stop fallback for all five properties

```sql
USE [Ski];

SELECT
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes,
    SkiBusStopDistance
FROM dbo.Property
WHERE PropertyId IN (2541, 1386, 36707, 33586, 2549)
ORDER BY PropertyId;
```

**Note:** This combined form is provided for quick reproduction. The individual fields/properties were verified in separate screenshots; a screenshot of this exact combined query was not supplied.

### Q04 — Find a six-minute / 0–1,000 m example

```sql
USE [Ski];

SELECT TOP (15)
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes
FROM dbo.Property
WHERE TownDistanceMinutes BETWEEN 6 AND 20
  AND TownDistance > 0
  AND TownDistance <= 1000
ORDER BY TownDistanceMinutes, PropertyId;
```

**Captured example:** Hotel Sportiv by VAYA `1386` / `600m` / `6min`.

### Q05 — Find properties more than 1,000 m from the centre

```sql
USE [Ski];

SELECT TOP (15)
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes
FROM dbo.Property
WHERE TownDistance > 1000
  AND TownDistanceMinutes > 0
ORDER BY TownDistance, PropertyId;
```

**Captured examples:** Apartment Petit Pont `33586` / `1100m` / `11min`; Rodella `2549` / `1200m` / `12min`.

### Q06 — Locate the bus-stop field in the Property schema

```sql
USE [Ski];

SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'Property'
  AND COLUMN_NAME LIKE '%Bus%';
```

**Captured result:** `SkiBusStopDistance` (`int`).

### Q07 — Investigate Apartment Petit Pont (missing bus-stop value)

```sql
USE [Ski];

SELECT
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes,
    SkiBusStopDistance
FROM dbo.Property
WHERE PropertyId = 33586;
```

**Captured result:** `33586` / `1100` / `11` / `NULL`.

### Q08 — Find properties above 1,000 m with a populated bus-stop distance

```sql
USE [Ski];

SELECT TOP (20)
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes,
    SkiBusStopDistance
FROM dbo.Property
WHERE TownDistance > 1000
  AND SkiBusStopDistance IS NOT NULL
  AND SkiBusStopDistance > 0
ORDER BY TownDistance, PropertyId;
```

**Captured candidates:** Bellevarde Lodge - Cognac `15174` / `1100m` / `11min` / bus `50m`; Rodella Hotel `2549` / `1200m` / `12min` / bus `10m`.

### Q09 — Inspect Rodella's fallback input values

```sql
USE [Ski];

SELECT
    PropertyId,
    Name,
    TownDistance,
    TownDistanceMinutes,
    SkiBusStopDistance
FROM dbo.Property
WHERE PropertyId = 2549;
```

**Captured result:** `2549` / `1200` / `12` / `10`.

### Q10 — Discover Feature tables and their columns

```sql
USE [Ski];

SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'dbo'
  AND (
       TABLE_NAME LIKE '%Feature%'
    OR TABLE_NAME LIKE '%Facility%'
    OR TABLE_NAME LIKE '%Amenit%'
    OR TABLE_NAME LIKE '%Attribut%'
  )
ORDER BY TABLE_NAME;

SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME IN ('Feature', 'FeatureLocated')
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

### Q11 — Verify Club Med property features by description

```sql
USE [Ski];

SELECT
    ft.FeatureTypeId,
    ft.Description,
    f.PropertyId,
    f.ResortId
FROM dbo.Feature AS f
INNER JOIN dbo.FeatureType AS ft
    ON f.FeatureTypeId = ft.FeatureTypeId
WHERE f.PropertyId = 36707
ORDER BY ft.FeatureTypeId;
```

**Captured result:** `FeatureTypeId=135`, **“Ski in/Ski out”**; other property features also returned.

### Q12 — Check the Snow Sure feature at Club Med's resort

```sql
USE [Ski];

SELECT
    ft.FeatureTypeId,
    ft.Description,
    f.PropertyId,
    f.ResortId
FROM dbo.FeatureType AS ft
LEFT JOIN dbo.Feature AS f
    ON ft.FeatureTypeId = f.FeatureTypeId
   AND f.ResortId = 25622
WHERE ft.Description LIKE '%Snow%'
ORDER BY ft.FeatureTypeId;
```

**Captured result:** `FeatureTypeId=88`, **“Snow sure”**, associated with Resort `25622`.

### Q13 — Optional schema discovery for Holiday ID relationships (proposed, NOT executed)

```sql
USE [Ski];

SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME IN ('HolidayId', 'PropertyId')
GROUP BY TABLE_SCHEMA, TABLE_NAME
HAVING COUNT(DISTINCT COLUMN_NAME) = 2
ORDER BY TABLE_NAME;
```

**Execution status:** Suggested for future investigation, but no result was captured in this QA session. **Do not use this query as evidence that any Holiday ID was linked to a property through SQL.**

## 7. Questions for Product / Development

1. **Accommodation-only offers:** Should the Transfers highlight still render as **“Not included”**, including a resort-level shortest transfer duration, when the booking has neither flights nor package transfers? If yes, which dataset supplies the time?
2. **Above 1,000 m:** Does the specification require a separate *Resort Location* card showing the ski-bus stop, or should that information appear under *Ski Access* instead? Rodella currently has the latter only.
3. **Missing values:** What is the required behaviour when `SkiBusStopDistance` or `TownDistanceMinutes` is `NULL`? Should the corresponding card be hidden, display an alternative message, or use another field?
4. **Exactly four highlights:** Are four cards mandatory for every property, or is the count a maximum depending on available/applicable data?
5. **Short-distance copy:** Should the under-six-minute message exactly read **“You're in the centre”**? Villa Cary currently shows **“In Resort Centre”**.

These questions are intended to settle acceptance criteria; they are not assertions about who implemented the underlying logic.

## 8. Remaining coverage / next test actions

- Resolve the questions above with the PO and/or developers, then rerun **Club Med** (transfers not included), **Rodella** (>1,000 m with bus-stop data), and **Petit Pont** (>1,000 m without bus-stop data).
- Test a resort **not** marked Snow Sure and confirm the correct alternative heading/height/cannon presentation.
- Test *Ski-in only* and *Ski-out only* separately, rather than relying exclusively on combined Ski-in/Ski-out examples.
- Verify a transfer-not-included offer with a confirmed resort fallback duration, if such a scenario is in scope.
- Attach UI and database screenshots to the Jira work items and record the expected-versus-actual outcome for each rerun.

## 9. Suggested concise Jira comment (UK English)

**Focus for reviewers:** the Transfers case is the clearest written-rule mismatch; the resort-location and missing-data cases require PO clarification before they are called bugs. The wording below preserves this distinction.

> **QA Validation — SKIP-2171 / SKIP-2301 | Staging**  
> Five property offers were reviewed, including database comparisons. Snow, ski access and included-transfer scenarios were observed working as expected.  
>  
> **Outstanding findings (linked to the ticket requirements):**  
> - **Club Med Les Arcs Panorama (Property 36707 / Holiday 203290784):** SKIP-2171 states **“IF No transfer is included THEN show 'Not included'”** and requires the shortest resort transfer time. The offer shows **“Transfers & Flights: Not included”**, but **no Transfers highlight or fallback time** appears (**2/4 cards**). Please confirm whether this rule also covers accommodation-only offers.  
> - **Rodella Hotel (Property 2549 / Holiday 158518187):** SKIP-2171 states **“IF the walk is over 1000m THEN distance to bus stop is shown instead”** under *Resort Centre Location*. SQL confirms **1,200 m** to the centre and a **10 m** bus stop, but the bus-stop information appears only under *Ski Access*; *Resort Location* is absent (**3/4 cards**). Please clarify the intended placement.  
> - **Apartment Petit Pont (Property 33586 / Holiday 127580406):** SKIP-2301 specifies **“4 primary highlights for property page displayed”**, whereas **1/4** appears. SQL records **1,100 m** to the centre and **`NULL`** for the bus-stop distance; the expected missing-data behaviour is not stated.  
>  
> **Result: IN TEST — clarification required before PASS.** SKIP-2301 specifies four highlights; please confirm whether the full set is mandatory when data or services are unavailable. Other conditional variants have not yet been fully tested.

---

**Report limitations:** UI observations, SQL screenshots and Jira text available in this QA session only. No exporter, API payload, network trace or application implementation was independently inspected for these tickets. SQL results do not establish which service populated the frontend. No SQL mutation is included or recommended in this report.
