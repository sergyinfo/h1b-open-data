# Data dictionary

All files: UTF-8, comma-separated, one header row, LF line endings. Empty cell =
not available (never `null`/`NA`). Ordering is deterministic and stable across
rebuilds.

## Conventions

**Fiscal year** — `fiscal_year` is the DOL fiscal year of the filing (Oct 1 –
Sep 30). FY2026 is partially reported.

**Wage annualisation** — `WAGE_RATE_OF_PAY_FROM` (the bottom of the offered
range) converted to annual USD by `WAGE_UNIT_OF_PAY`:

| Unit | Multiplier |
| --- | --- |
| Year | × 1 |
| Hour | **× 2080** (40 h/week × 52 weeks) |
| Week | × 52 |
| Bi-weekly | × 26 |
| Month | × 12 |

Only annualised wages in **[$20,000, $1,000,000]** are included
in wage statistics; anything outside is treated as a data-entry error and
excluded. Filings with no parseable wage still count toward `filings` but not
toward wage columns. **All wage outputs are rounded to the nearest $100.**

**Percentiles** — computed in PostgreSQL with
`percentile_cont(q) WITHIN GROUP (ORDER BY annual_wage)`, i.e. linear
interpolation between adjacent values (not nearest-rank). `wage_median` is
`percentile_cont(0.5)`.

**Certification percentages** — share of *all* filings for that row's entity:

    certified_pct      = 100 × count(CASE_STATUS = 'Certified')           / total_filings
    cert_withdrawn_pct = 100 × count(CASE_STATUS = 'Certified - Withdrawn')/ total_filings
    withdrawn_pct      = 100 × count(CASE_STATUS = 'Withdrawn')           / total_filings
    denied_pct         = 100 × count(CASE_STATUS = 'Denied')              / total_filings

Reported to 2 decimal places. These four need not sum to 100: filings in other
or pending states are counted in the denominator only.

**Small-count suppression (MIN_N = 25)** — any row whose underlying filing
count is below 25 is **omitted entirely** (not blanked). Applied at every
grain, so totals do not reconcile exactly against national figures. This keeps
every published row a genuine aggregate.

**Employers** — keyed on the *canonical* employer (name variants already
resolved). `employer_slug` matches the URL at https://h1b.report/employer/<slug>.
The variant→canonical mapping, FEIN, and any row-level filing data are **not**
part of this release.

---

## national_by_year.csv

One row per fiscal year.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `fiscal_year` | integer | FY | DOL fiscal year |
| `total_filings` | integer | filings | `count(*)` of LCA filings that year |
| `certified_pct` | decimal(2dp) | % | Certified ÷ total_filings × 100 |
| `median_wage` | integer | USD/yr | `percentile_cont(0.5)`, rounded to $100 |

## occupation_by_year.csv

One row per (SOC code × fiscal year). Ordered by `soc_code`, `fiscal_year`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `soc_code` | string | — | SOC occupation code, e.g. `15-1252` |
| `soc_title` | string | — | SOC occupation title |
| `fiscal_year` | integer | FY | DOL fiscal year |
| `filings` | integer | filings | Filings for that SOC that year |
| `wage_p25` | integer | USD/yr | 25th percentile annualised wage |
| `wage_median` | integer | USD/yr | 50th percentile |
| `wage_p75` | integer | USD/yr | 75th percentile |

## occupation_by_state.csv

One row per (SOC code × worksite state). Ordered by `soc_code`, `state`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `soc_code` | string | — | SOC occupation code |
| `soc_title` | string | — | SOC occupation title |
| `state` | string | — | 2-letter **worksite** state code (not employer HQ) |
| `filings` | integer | filings | Filings for that SOC in that state |
| `wage_median` | integer | USD/yr | 50th percentile annualised wage |

## occupation_wage_by_level.csv

One row per (SOC code × prevailing-wage level). Ordered by `soc_code`, `pw_level`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `soc_code` | string | — | SOC occupation code |
| `soc_title` | string | — | SOC occupation title |
| `pw_level` | string | — | `I`–`IV`: DOL prevailing-wage skill level. I ≈ entry, IV ≈ fully competent/senior. Filings without a level are excluded from this file. |
| `n` | integer | filings | Filings at that SOC and level |
| `wage_p25` | integer | USD/yr | 25th percentile annualised wage |
| `wage_median` | integer | USD/yr | 50th percentile |
| `wage_p75` | integer | USD/yr | 75th percentile |

## state_by_year.csv

One row per (worksite state × fiscal year). Ordered by `state`, `fiscal_year`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `state` | string | — | 2-letter worksite state code |
| `fiscal_year` | integer | FY | DOL fiscal year |
| `filings` | integer | filings | Filings with a worksite in that state |
| `wage_median` | integer | USD/yr | 50th percentile annualised wage |

## sector_by_year.csv

One row per (NAICS-2 sector × fiscal year). Ordered by `naics2`, `fiscal_year`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `naics2` | string | — | First 2 digits of the employer's NAICS code |
| `sector_name` | string | — | NAICS sector label, e.g. `Professional, Scientific & Technical Services` |
| `fiscal_year` | integer | FY | DOL fiscal year |
| `filings` | integer | filings | Filings in that sector that year |
| `wage_median` | integer | USD/yr | 50th percentile annualised wage |

## employers_summary.csv

One row per canonical employer with ≥ 25 lifetime filings. Ordered by
`total_filings` descending, then `employer_name`.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `employer_slug` | string | — | URL slug; `https://h1b.report/employer/<slug>` |
| `employer_name` | string | — | Canonical employer name |
| `employer_state` | string | — | 2-letter state on the employer record (HQ, not worksite) |
| `total_filings` | integer | filings | Lifetime filings across all years |
| `first_fy` | integer | FY | Earliest fiscal year with a filing |
| `last_fy` | integer | FY | Latest fiscal year with a filing |
| `certified_pct` | decimal(2dp) | % | See "Certification percentages" above |
| `cert_withdrawn_pct` | decimal(2dp) | % | Certified then withdrawn |
| `withdrawn_pct` | decimal(2dp) | % | Withdrawn before decision |
| `denied_pct` | decimal(2dp) | % | Denied |
| `median_wage` | integer | USD/yr | 50th percentile annualised wage, all years |
| `top_soc_code` | string | — | Most-filed SOC for that employer (ties broken by lowest `soc_code`) |
| `top_soc_title` | string | — | Title of `top_soc_code` |

## soc_coverage_by_year.csv

Diagnostic table: how much of each fiscal year carries a SOC occupation code.
Read this before comparing occupation counts across years — a year with low
coverage will under-report every SOC.

| Column | Type | Unit | Definition |
| --- | --- | --- | --- |
| `fiscal_year` | integer | FY | DOL fiscal year |
| `total_filings` | integer | filings | All filings that year |
| `filings_with_soc_code` | integer | filings | Filings carrying a SOC code — the population behind the `occupation_*` files |
| `soc_coverage_pct` | decimal(2dp) | % | `filings_with_soc_code` ÷ `total_filings` × 100 |

Not subject to MIN_N suppression: it is a per-year completeness diagnostic, and
every row aggregates an entire fiscal year.

## sample_occupation_by_state.csv

First 100 rows of `occupation_by_state.csv`, same schema — a preview only.
