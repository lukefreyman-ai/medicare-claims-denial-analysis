# Medicare Claims Denial Risk Analysis

Applying CMS National Correct Coding Initiative (NCCI) edits to 780,000 Medicare 
outpatient claims to identify billing patterns that would trigger payer denials, 
and quantifying the dollars at risk.

**Result:** 31,693 claims (4.1%) contain at least one hard-edit violation. 41% of 
all violations trace to a single code pair. Violations are systemic across 
providers rather than concentrated in outlier facilities.

---

## Why this exists

No public dataset contains denied claims — denial data is proprietary to payers 
and providers. This project works around that by taking *paid* claims and applying 
CMS's published billing rules to identify which payments should not have been made.

This is the core logic behind two real product categories: **payment integrity** 
(payers recovering improper payments) and **claim scrubbing** (providers preventing 
audit exposure).

## Data sources

Neither dataset is redistributed in this repo. Download separately:

- **Claims:** [CMS DE-SynPUF](https://www.cms.gov/data-research/statistics-trends-and-reports/medicare-claims-synthetic-public-use-files) — Outpatient Claims, Sample 1 (790,790 rows, Dec 2007–Dec 2010, $224.5M in payments)
- **Rules:** [CMS NCCI Hospital PTP Edits](https://www.cms.gov/medicare/coding-billing/ncci-medicare) — 2026 Q3 (v322r0), 1,864,729 code pairs across four files

> NCCI files contain AMA-licensed CPT codes and require accepting a license 
> agreement at download. They are excluded from this repo.

## Method

1. Load 790,790 outpatient claims; deduplicate multi-segment claims to 779,815 unique
2. Normalize wide-format claims (45 procedure columns) into 3,774,339 service lines
3. Load full NCCI PTP edit file; filter to modifier indicator 0 (64,711 hard edits 
   permitting no override)
4. Generate all code-pair combinations per claim (3,558,615 candidate pairs)
5. Match bidirectionally against the edit table — NCCI pairs are directional
6. Aggregate violations by code pair, provider, and dollar exposure

## Findings

**#1 — Initial hypothesis not supported.** Hypothesized that venipuncture (36415) 
billed with lab panels was an NCCI violation. Screened 74,997 claims / $17.6M — then 
verified against the full edit file and found the pair does not exist in either 
direction. Both codes appear in the tables, so the negative result is real, not a 
matching error. The relationship is governed by Clinical Lab Fee Schedule payment 
policy, not PTP edits. Hypothesis withdrawn.

**#2 — Duplicate metabolic panels (confirmed).** 80053 (comprehensive) billed with 
80048 (basic), modifier indicator 0. The basic panel is a subset of the comprehensive. 
15,729 claims / $5.1M.

**#3 — Full hard-edit scan.** 38,350 violating pairs across 31,693 claims (4.1%). 
$12,674,090 in claims touched; **~$1,222,331 estimated line-level exposure** after 
correcting for the fact that a violating claim contains a median of 11 service lines, 
typically one of which is affected. Violations concentrate heavily by code — 80048/80053 
alone accounts for 41% — and cluster by department: laboratory, physical therapy, radiology.

**#4 — Systemic, not provider-concentrated.** 4,247 of 6,293 providers (67.5%) have at 
least one violation, but the top 10 account for only 6.4%. Provider-level auditing would 
be inefficient; the pattern points to shared billing software defaults and order-set 
design rather than facility-specific behavior.

**External validation.** CMS's CERT program reported a 6.55% Medicare FFS improper 
payment rate for FY2025 ($28.83B). This analysis observed 4.1%, detecting only hard PTP 
code-pair conflicts — a narrow subset of CERT's scope, which also includes coverage, 
medical necessity, and documentation failures. A result below CERT is the expected 
relationship; a result above it would have indicated methodological error.

## Limitations

- **Synthetic data.** DE-SynPUF is statistically generated. Violation rates hold for this 
  dataset; they should not be assumed to match real Medicare claims. Provider distribution 
  in particular may reflect synthesis artifacts.
- **Binned payments.** Values appear capped at $3,300, so all dollar figures are approximate.
- **Line-level exposure is estimated, not measured.** DE-SynPUF reports payment only at 
  claim level. Exposure is estimated by even allocation across lines — an approximation, 
  since lab codes typically run below the claim average.
- **Temporal mismatch.** 2026 Q3 edits applied to 2007–2010 claims. CMS publishes only the 
  current quarter; historical edit files are not publicly archived. Core bundling 
  relationships are stable, but production use would version-match edits to service dates.
- **No modifier data.** DE-SynPUF contains no modifiers, so modifier-indicator-1 edits 
  (allowed with documentation) cannot be evaluated. Analysis is therefore restricted to 
  indicator-0 edits.

## Reproducing

```bash
pip install pandas openpyxl
```

Place downloaded files in `data/`, then run `notebooks/Claim-analysis.ipynb` top to bottom.

## Next steps

- MUE (Medically Unlikely Edits) scan — maximum units per code per day
- Appeal prioritization: rank violations by expected recovery value
- Patient-level segmentation using the DE-SynPUF Beneficiary Summary File
- Streamlit interface for file upload and violation reporting

---

Luke Freyman · [LinkedIn](https://linkedin.com/in/luke-freyman-a22ab7249)
