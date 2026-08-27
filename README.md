# H-1B / LCA Open Data — aggregate tables (FY2010–FY2026)

Aggregate statistics on **9,331,644 US Labor Condition Applications** (LCAs) —
the wage-and-worksite filing every employer must submit to the Department of Labor
before sponsoring an H-1B, H-1B1, or E-3 worker. Covers **126,466 employers**
and **782 occupations** across fiscal years 2010–2026.

> **This is a mirror.** The canonical home of this dataset — always the newest
> release, with the data dictionary and license — is
> **<https://h1b.report/open-data>**. Mirrors are refreshed at least one
> quarterly release behind the canonical page. The same data is also served
> interactively (charts, rankings, per-employer analytics) at https://h1b.report.

## Source

US Department of Labor, Office of Foreign Labor Certification (OFLC) —
quarterly LCA disclosure files. Public domain at source.

## What's here

| File | Rows | Grain |
| --- | --: | --- |
| `national_by_year.csv` | 17 | fiscal year |
| `occupation_by_year.csv` | 4,477 | SOC × fiscal year |
| `occupation_by_state.csv` | 6,578 | SOC × worksite state |
| `occupation_wage_by_level.csv` | 1,132 | SOC × prevailing-wage level |
| `state_by_year.csv` | 931 | state × fiscal year |
| `sector_by_year.csv` | 425 | NAICS-2 sector × fiscal year |
| `employers_summary.csv` | 18,927 | canonical employer |
| `soc_coverage_by_year.csv` | 17 | fiscal year (coverage diagnostic) |

`sample_occupation_by_state.csv` is a 100-row preview. Column-level definitions
are in [`data_dictionary.md`](./data_dictionary.md).

## How it's cleaned

Raw OFLC files are inconsistent across years — the schema changed in FY2020 (the
FLAG release), employer names are free text, and wages are quoted hourly, weekly,
bi-weekly, monthly, or yearly. This release is derived from a normalised corpus:

- **Canonical employers.** Name variants ("IBM Corp.", "I.B.M. Corporation",
  "International Business Machines") are resolved to a single canonical employer,
  so an employer's filings aren't scattered across spellings. Aggregates key on
  the canonical employer; the underlying name-variant mapping is not published.
- **SOC occupation codes** are normalised to a single taxonomy, with job titles
  matched against the BLS Direct Match Title File and a classifier for the rest.
- **Annualised wages.** `WAGE_RATE_OF_PAY_FROM` is converted to annual USD
  (hourly × 2080, weekly × 52, bi-weekly × 26, monthly × 12).

## Caveats — please read before citing

- **An LCA is not a visa.** It is a prerequisite filing. It is not an H-1B
  petition, not an approval, and not a hire. One filing can cover multiple
  positions, and many filings never become petitions. Filing counts are **not**
  visa counts.
- **Wages are offered wages, not take-home pay.** `WAGE_RATE_OF_PAY_FROM` is the
  bottom of the offered range on the filing. It excludes bonus and equity, and it
  is not verified against what was actually paid.
- **FY2026 is partially reported.** It currently holds 210,387 filings against 596,552 in FY2025 (65% lower), which reflects reporting lag, not a real drop. DOL publishes quarterly, so the
  most recent fiscal year is the least complete and its totals will rise in later
  releases. It is left uncorrected here — do not read it as a real decline.
- **A small share of filings carry no occupation code.** 4.7% of filings (443,000) carry no SOC code. This is a flat background rate, not a gap in particular years: per-year coverage runs 94.5%-95.9%, so years remain comparable with each other. The three
  `occupation_*` files only include filings that carry a SOC code. The national,
  state, and sector files are unaffected — they count every filing. Per-year
  coverage is published in `soc_coverage_by_year.csv`.
- **Small cells are suppressed.** Any row backed by fewer than 25 filings is
  omitted, so columns do not sum exactly to national totals.
- **Wage outliers are excluded.** Only wages between $20,000 and
  $1,000,000 annualised are counted, dropping data-entry errors.
- Percentages are of *all* filings for that entity, including still-pending ones.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to share and
adapt, including commercially, **with attribution**. See [`LICENSE`](./LICENSE).

## Citation

> h1b.report (2026). *H-1B / LCA Open Data: aggregate tables,
> FY2010–FY2026.* Derived from US DOL OFLC LCA disclosure data. https://h1b.report
