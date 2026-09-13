# Interview guide - medicare-claims-denial-analysis

## What it does, in three sentences

It takes 780,000 paid Medicare outpatient claims (CMS DE-SynPUF, synthetic public-use file) and applies CMS's own
published billing rules - the NCCI procedure-to-procedure edits - to find code pairs that should never be billed
together. It found that 31,693 claims (4.1 %) contain at least one hard-edit violation, worth about $1.2 million in
line-level exposure, with 41 % of violations coming from a single lab panel pair. Because the violations are spread
across 67 % of providers, the finding is that the problem is systemic (billing-software defaults), not a few bad actors.

## Why each key decision was made (plain language)

- **Paid claims + published rules instead of a "denials dataset".** No public dataset has denials. Applying the rules
  that cause denials to paid claims is exactly how payment-integrity vendors find recoverable overpayments.
- **Modifier-indicator-0 edits only.** Those are the pairs where no modifier can justify billing both codes, so a hit
  is unambiguous. DE-SynPUF has no modifier data, so indicator-1 edits could not be judged fairly.
- **Bidirectional matching.** NCCI pairs are directional (column 1 / column 2); checking both orders avoids missing
  hits and avoids double-counting.
- **Line-level exposure estimate.** DE-SynPUF reports payment per claim. A violating claim has a median of 11 lines, so
  claiming the whole $12.7 million would overstate exposure ~10x; the $1.2 million figure allocates payment evenly.
- **Withdrawing the first hypothesis.** Venipuncture with lab panels looked like a violation; the edit file says it
  isn't. Keeping that negative result in the README shows the analysis was checked against the source, not assumed.

## What I'd do differently

- Version-match edit files to service dates (2026 edits were applied to 2007-2010 claims).
- Add Medically Unlikely Edits (units per code per day) - a second, independent class of rule.
- Move the notebook into a package with tests, and add a Streamlit upload-and-scan interface.
- Segment by beneficiary using the summary file to see whether violations cluster by patient profile.

## Five questions an interviewer would ask, with honest answers

**1. The data is synthetic - do the numbers mean anything?**
The *rates* do not transfer to real Medicare; the *method* does. CMS's own CERT program reports a 6.55 % improper
payment rate across all causes; finding 4.1 % from one narrow cause (hard code-pair conflicts) is the right order of
magnitude, and finding more than CERT would have signalled a bug.

**2. Why does one code pair account for 41 %?**
80053 (comprehensive metabolic panel) billed with 80048 (basic metabolic panel). The basic panel is a subset of the
comprehensive one, so billing both is double-billing by definition. It is common because order sets and lab
interfaces add both automatically - which is also why the violations are spread across most providers.

**3. How would a payer use this?**
Pre-payment: as a claim-scrubbing rule that rejects or pends the claim. Post-payment: to prioritise recovery by expected
dollars. The dollars-at-risk ranking is what makes it actionable rather than a list of errors.

**4. What is the biggest source of error in the $1.2 million figure?**
Even allocation of payment across lines. Lab lines are usually cheaper than the claim average, so the true exposure is
probably lower. It is a bound, not a measurement, and the README says so.

**5. Why NCCI rather than a machine-learning model of denials?**
Because the rules are public, deterministic and defensible in an appeal. A model would need labelled denials, which do
not exist publicly, and would still have to explain itself in terms of these rules.

## Numbers to have in your head

790,790 claims -> 779,815 unique -> 3.77 M service lines · 64,711 indicator-0 edit pairs · 3.56 M candidate pairs
checked · 38,350 violating pairs on 31,693 claims (4.1 %) · $12.7 M claims touched, ~$1.22 M line-level exposure ·
80048/80053 = 41 % of violations · 67.5 % of providers have >= 1 violation, top 10 providers = 6.4 % · CERT FY2025 6.55 %.
