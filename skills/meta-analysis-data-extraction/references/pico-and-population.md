# Precise PICO and Population Accounting

Read this reference whenever the skill is invoked. It governs question-level PICO, study-level PICO mapping, and intervention/exposure population counts.

## 1. Question-level PICO

Parse only what the user's wording supports. Do not silently broaden a disease, treatment, comparator, setting, age group, follow-up period, or outcome.

Create `Question_PICO` with one row per element:

| Element | Required content |
|---|---|
| `P` | Population, condition, stage/severity, age constraints, setting, and review-relevant inclusion/exclusion qualifiers explicitly present in the question |
| `I` | Exact intervention or exposure, including dose, route, duration, timing, or exposure definition when stated |
| `C` | Exact active, placebo, usual-care, non-exposed, alternative-dose, or other comparator; use value `NR` and status `Not specified` when absent from the question |
| `O` | Each outcome as a separate row, preserving definitions and priority when stated |
| `T` | Follow-up or time horizon when stated; otherwise value `NR` and status `Not specified` |
| `S` | Study-design restriction when stated; otherwise value `NR` and status `Not specified` |

For every element record:

- `Question_Text_Evidence`: the shortest exact phrase from the user's question that supports it.
- `Normalized_Concept`: a faithful standardized label, not a broader synonym.
- `Status`: `Explicit`, `Implicit`, `Not specified`, or `Ambiguous`.
- `Confidence`: `High`, `Moderate`, or `Low`.
- `Rationale_or_Ambiguity`: concise reasoning only when needed.

Also create one normalized Target PICO record with:

`Target_Population | Target_Intervention_or_Exposure | Target_Comparator | Target_Outcomes | Target_Effect_Measure | Target_Followup | Target_Study_Design`

Do not infer any sample size at this stage. Preserve multiple requested outcomes separately in downstream analytic rows. The Target PICO is the relevance and eligibility reference for all supplied studies.

When wording permits more than one materially different interpretation, choose the most literal primary interpretation, list alternatives, mark `Ambiguous`, and avoid using the ambiguous element as a hard exclusion criterion without user confirmation.

## 2. PICO-to-study group mapping

After reading each PDF, preserve the Target PICO and the study's actual group definitions separately. Cite `Source_Type`, `Source_ID`, and page for the study definitions.

Create:

| Study_ID | PICO_Target_Intervention | PICO_Target_Comparator | Study_Reported_Exposure | Study_Reported_Comparator | Intervention_Match | Comparator_Match | Source_Type | Source_ID | Page | Notes |
|---|---|---|---|---|---|---|---|---|---:|---|

Use only these match values:

- `EXACT`: the study group directly equals the Target PICO group.
- `SUBGROUP`: the Target PICO group is explicitly identifiable and separately reported within a broader study group.
- `PARTIAL`: the study group overlaps the Target PICO but the target group cannot be completely isolated.
- `NO`: the study group does not correspond to the Target PICO.

Do not declare a match from title/abstract wording when the full PDF provides a more specific definition. Do not force an observational exposure into a treatment interpretation.

### Comparison alignment and eligibility

For every candidate analytic row create `Comparison_Alignment`:

- `EXACT`: the reported effect directly represents the Target PICO comparison.
- `SUBGROUP_WITH_FULL_OUTCOME_DATA`: the target comparison is a subgroup of a broader study population and the PDF directly reports all necessary target-subgroup outcome/effect data.
- `SUBGROUP_N_ONLY`: target-subgroup N is reported, but the outcome/effect is only for a broader comparison.
- `PARTIAL`: the reported comparison overlaps but does not fully represent the Target PICO.
- `NOT_ELIGIBLE`: the reported comparison does not represent the Target PICO.

Create `Eligible_For_Primary_Meta_Analysis` with `YES`, `NO`, or `UNCERTAIN`. Normally map:

- `EXACT` → `YES`
- `SUBGROUP_WITH_FULL_OUTCOME_DATA` → `YES`
- `SUBGROUP_N_ONLY` → `NO`
- `PARTIAL` → `NO` or `UNCERTAIN`, depending on whether the protocol resolves the overlap
- `NOT_ELIGIBLE` → `NO`

A matching N alone never makes a row eligible. Population, actual comparison, outcome, effect estimate, and denominator must refer to the same analysis.

## 3. Intervention/exposure arm identity

Build an explicit arm map before entering counts:

| Study_ID | Original_Arm_Label | Standardized_Arm_Label | Arm_Role | Intervention_or_Exposure_Definition | Dose_or_Intensity | Timing_or_Duration | Source_ID | Page |
|---|---|---|---|---|---|---|---|---:|

`Arm_Role` is `Intervention`, `Exposure`, `Comparator`, `Control`, or `Other`. Preserve every multi-arm group separately. Do not swap article group order during raw reconstruction; standardized roles are added only in normalized tables.

### Study-reported and PICO-aligned sample sizes

Maintain both truths:

- `Study_Reported_Exposure_N`
- `Study_Reported_Comparator_N`
- `PICO_Aligned_Intervention_N`
- `PICO_Aligned_Comparator_N`

The study-reported fields describe the study's named groups. The PICO-aligned fields contain a directly reported target-specific group or subgroup count. Selecting a directly reported target-specific N is extraction, not calculation. Do not obtain it by percentages, subtraction, residual counts, or assumptions.

Never replace the study-reported N with the PICO-aligned N. For example, if a study reports a broad comparator N of 4,500 and directly reports the target comparator subgroup N as 902, retain both 4,500 and 902 in their respective fields.

### Effect-estimate population lock

For every effect estimate record:

- `Effect_Exposure`
- `Effect_Comparator`

These are the actual groups represented by the HR, RR, OR, IRR, RD, MD, SMD, or other estimate. If an estimate compares an intervention with a broad comparator, it remains attached to that broad comparator even when a target-comparator subgroup N is separately reported. Never relabel a broad comparison as the target comparison without a directly reported target-specific effect estimate.

## 4. Population-flow accounting

The phrase “intervention number” is not a single universal denominator. Extract the exact reported count at each relevant stage into `Population_Flow`:

| Study_ID | Arm | Arm_Role | Population_Stage | Analysis_Population | Outcome | Followup_Value | Followup_Unit | Reported_N | Source_Type | Source_ID | Page | Extraction_Confidence | Notes |
|---|---|---|---|---|---|---:|---|---:|---|---|---:|---|---|

Use the narrowest supported `Population_Stage`, such as:

- `Screened`
- `Eligible`
- `Randomized`
- `Assigned`
- `Received_Intervention`
- `Exposed`
- `Baseline`
- `Safety_Analysis`
- `ITT_Analysis`
- `Modified_ITT_Analysis`
- `Per_Protocol_Analysis`
- `Outcome_Analysis`
- `Matched`
- `Weighted`
- `Effective_Sample_Size`
- `Followup_Assessed`
- `Risk_Set`
- `Other`

Record `Analysis_Population` exactly as the study identifies it. If the study reports only a total, record `Arm = Total`, set arm-specific counts to `NR`, and do not allocate the total across arms.

### Denominator safeguards

- Never use screened, eligible, assigned, treated, baseline, or followed counts interchangeably.
- Never use `Number_at_Risk` as baseline N, event count, person-years, censoring count, or analysis N. Risk sets are time-specific.
- Never use events or person-years as participant N.
- Never copy baseline N into an outcome-specific denominator unless the article explicitly establishes they are identical.
- Keep randomized/assigned counts separate from treated/as-treated counts and ITT separate from per-protocol.
- For cluster trials, distinguish number of clusters from number of participants; include `Count_Unit = clusters` or `participants` when extending the table.
- For crossover studies, distinguish unique participants from observations or periods.
- Weighted pseudo-population counts may be decimal and are not actual participant counts. Do not call them unweighted N.
- Record an effective sample size only when directly reported and label it `Effective_Sample_Size`.
- Matched-pair counts, matched-person counts, and matching ratios are distinct. Preserve their units and definitions.
- If an analysis excludes participants after baseline, use the reported analysis denominator for that analysis row and retain the baseline count separately in `Population_Flow`.
- When multiple outcomes have different denominators, create outcome-specific population-flow rows.

For the normalized master row, populate the most specific count fields supported by the corresponding analysis. If the proper mapping is uncertain, use `UNCERTAIN`, keep the candidate source values in `Population_Flow`, and require manual verification rather than guessing.
