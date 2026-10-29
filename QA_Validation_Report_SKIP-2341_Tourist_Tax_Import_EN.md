# QA Validation Report | SKIP-2341 — CMA > Fee List

**Ticket:** SKIP-2341 — CMA > Fee list  
**Related epic:** SKIP-2352 — CMA Pricing Transparency  
**Environment:** Iglu Ski Staging — SQL Server, `Ski` database  
**Validation evidence date:** 28 October 2026  
**Test approach:** Read-only database validation, reconciliation with the source **FINAL SHEET**, and targeted sampling  
**QA result:** **PASS — Database import validation successful (sample-based)**

> **Evidence standard:** The result applies to the import checks described below. It does not represent a full row-by-row reconciliation of all 707 records or prove downstream importer, exporter, pricing or search-card behaviour. SQL and spreadsheet screenshots were reviewed during the QA session; they are not embedded in this Markdown file.

---

## 1. Objective and acceptance criterion

The ticket requests importing the resort tourist-tax fee list into the database, using the **FINAL SHEET** tab of the supplied *Tourist tax* spreadsheet as the reference.

**Acceptance criterion (Jira):** “Import fee list”.

The QA objective was to verify that imported data existed in `Ski.dbo.TouristTax`, that selected resort rates, accommodation classifications and currencies agreed with the reference spreadsheet, and that there were no duplicate entries for the checked classification key.

## 2. Validation summary

| Check | Evidence / observed result | Status |
| --- | --- | --- |
| Import volume | `SELECT COUNT(*)` returned **707** records; the attached SQL import script identifies records **1–707**. | **PASS** |
| Selected source-to-database comparisons | Five resort IDs compared against **FINAL SHEET**: `1601`, `1602`, `1603`, `1941`, `25598`. The sampled rates and classification values matched. | **PASS — sampled** |
| Currency | EUR records confirmed for the selected French and Italian resorts; CHF confirmed for Lenzerheide. | **PASS — sampled** |
| Tax rate unit | Returned rows used **“Per person, per night”**, consistent with the selected reference entries. | **PASS — sampled** |
| Duplicate classification keys | A full-table grouping by `ResortId`, `AccommodationClassificationTypeId` and `AccommodationClassification` returned **no duplicate groups**. | **PASS** |

## 3. Verified sample register

The following records were inspected in the database and compared with the visible **FINAL SHEET** source values.

| Resort ID | Resort | Country / currency | Examples of matched classifications and rates |
| --- | --- | --- | --- |
| `1601` | Chamonix | FR / EUR | 2★ €1.00; 3★ €1.76; 4★ €3.63; 5★ €5.06; Apartments (`APA`) €1.60 |
| `1602` | Alpe d'Huez | FR / EUR | 2★ €1.00; 3★ €2.06; 4★ €3.15; 5★ €4.37 |
| `1603` | La Plagne | FR / EUR | 3★ €1.93; 4★ €3.03; 5★ €3.96; Apartments (`APA`) €2.07 |
| `1941` | Lenzerheide | CH / CHF | Resort-wide (`All`) CHF 4.50 |
| `25598` | Val di Fassa | IT / EUR | 2★ €1.50; 3★ €2.00; 4★ €2.50; 5★ €3.00 |

**Observed outcome:** No discrepancy was identified for the sampled resort records, tax rates, currency codes or classifications shown in the evidence.

## 4. SQL evidence — reproducible read-only checks

**Q01 — Confirm imported record count**

```sql
SELECT COUNT(*) AS TotalRecords
FROM [Ski].[dbo].[TouristTax];
```

**Observed:** `707` records. This matches the import script's stated insert range; it is not a claim that all rows in the original spreadsheet were imported.

**Q02 — Inspect selected source-to-database records**

```sql
SELECT
    ResortId,
    Name,
    CountryCode,
    AccommodationClassificationTypeId,
    AccommodationClassification,
    TouristTaxRate,
    CurrencyCode,
    RateUnit
FROM [Ski].[dbo].[TouristTax]
WHERE ResortId IN (1601, 1602, 1603, 1941, 25598)
ORDER BY ResortId, AccommodationClassification;
```

**Observed:** Sampled records matched the **FINAL SHEET** for the five resorts, including star-rating categories, apartment classifications, rates, and EUR/CHF currency codes.

**Q03 — Detect duplicate classification entries across the entire table**

```sql
SELECT
    ResortId,
    AccommodationClassificationTypeId,
    AccommodationClassification,
    COUNT(*) AS Total
FROM [Ski].[dbo].[TouristTax]
GROUP BY
    ResortId,
    AccommodationClassificationTypeId,
    AccommodationClassification
HAVING COUNT(*) > 1
ORDER BY ResortId, AccommodationClassification;
```

**Observed:** No rows returned. No duplicates were identified using this defined composite classification key.

## 5. Coverage boundaries and QA considerations

- The **707-record count** confirms the current table population and agrees with the import script's stated volume; it is not a complete source-versus-target reconciliation.
- The value-level verification covered **five selected resorts**, rather than every resort in the **FINAL SHEET**.
- The source spreadsheet includes entries such as `#N/A`, blank fields or `None` in other locations. Their intended import handling was **not fully verified** by the five-resort sample.
- This ticket's check does **not** establish that tourist tax is correctly calculated or propagated through `OptimisedSearch`, importer/exporter pipelines, holiday indexes or search cards. Those are separate work items and require their own evidence.
- The absence of duplicate classification keys is not a substitute for verifying business priority rules, such as whether resort-wide values take precedence over property-type or star-rating values.

## 6. Final QA assessment

**Result: PASS — Database import validation successful (sample-based).**

The imported table contained **707 records**; the five sampled resorts matched their **FINAL SHEET** rates, classifications and currencies; and the full-table duplicate-key check returned no results. No discrepancy was identified within the performed checks.

This is a **PASS for the tested fee-list import acceptance criterion at database level**, not a blanket certification of every row or any downstream tourist-tax functionality.

## 7. Suggested Jira comment (UK English)

> **QA Validation | Staging — SKIP-2341**
>
> Validated the tourist tax import in `Ski.dbo.TouristTax` against the **FINAL SHEET** using read-only SQL queries.
>
> - **Record count:** `SELECT COUNT(*)` confirmed **707 records**, consistent with the import script.
> - **Data accuracy:** Checked five resorts using `WHERE ResortId IN (1601, 1602, 1603, 1941, 25598)`. Sampled tax rates, accommodation classifications and EUR/CHF currency codes matched the source.
> - **Duplicates:** `GROUP BY ResortId, AccommodationClassificationTypeId, AccommodationClassification HAVING COUNT(*) > 1` returned no duplicate groups across the table.
>
> No discrepancies were identified within the performed checks. Detailed SQL evidence and sampling scope are documented in the attached QA report.
>
> **Result: PASS — Database import validation successful (sample-based).**
