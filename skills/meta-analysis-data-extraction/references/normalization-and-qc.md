# Numerical Normalization, Selection, and Quality Control

Apply these rules only after original tables and figures have been reconstructed and the meaning of each value has been established from headers, legends, footnotes, or Methods.

## Atomic decomposition

### Effect estimates

For a directly reported `0.74 (0.60–0.93)` identified as an HR with 95% CI:

- `Effect_Measure = HR`
- `Effect_Estimate = 0.74`
- `CI_Level = 95`
- `CI_Lower = 0.60`
- `CI_Upper = 0.93`

Do not retain the combined string in a normalized numeric field; keep it in `Raw_Reported_Value` or the reconstruction.

Standardize synonymous effect labels such as “hazard ratio” to `HR`, while expressing crude versus adjusted in `Analysis_Model` and `Adjusted`. Do not create meaningless fields such as `HR_1`, `HR_2`, or `HR_3`.

### Cases and person-years

Split a column explicitly labeled `Cases/PYs`, such as `133/38273`, into `Events = 133` and `Person_Years = 38273`. Never put person-years in N.

### Count and percent

Split `133 (12.4%)` into `Count = 133` and `Percent = 12.4`. Do not include `%` in the percent field.

### Mean and SD

Split `65.4 (9.45)` into `Mean = 65.4` and `SD = 9.45` only if the table, footnote, legend, or Methods explicitly defines the format as mean (SD). Otherwise keep the raw string and mark its components `UNCERTAIN` or `NR` as appropriate.

### Median and IQR

Split `2.8 (1.2–4.3)` into `Median = 2.8`, `IQR_Lower = 1.2`, and `IQR_Upper = 4.3` only when explicitly defined as median (IQR).

### P values

Preserve the reported relation: `p < 0.001` becomes `P_Value = <0.001`. Do not convert it to `0.000999` or another invented value.

## Classification safeguards

### Sample size

Use the most specific supported fields:

- `Exposure_Unweighted_N`, `Comparator_Unweighted_N`
- `Exposure_Weighted_N`, `Comparator_Weighted_N`
- `Exposure_Matched_N`, `Comparator_Matched_N`
- `Exposure_Analysis_N`, `Comparator_Analysis_N`

If only total N is reported, use `Total_N` and set arm-specific fields to `NR`. Do not copy totals into arms. Reconcile every analytic-row N with `Population_Flow` and its population stage.

Also preserve `Study_Reported_Exposure_N` and `Study_Reported_Comparator_N` separately from `PICO_Aligned_Intervention_N` and `PICO_Aligned_Comparator_N`. A directly printed target-subgroup N may populate a PICO-aligned field without calculation, but it cannot replace the broad study-group N or be paired with a broad-group effect estimate.

### Events

Keep `Raw_Events`, `Weighted_Events`, `Matched_Events`, and `Adjusted_Events` distinct. An IPTW or adjusted effect estimate does not make a raw event count weighted or adjusted. Populate a weighted/adjusted event field only when explicitly reported as such.

### Person-years

Keep `Raw_PersonYears`, `Weighted_PersonYears`, `Matched_PersonYears`, and `Adjusted_PersonYears` distinct. Never copy raw person-years into a weighted field merely because the effect model uses weighting.

### Follow-up, outcomes, comparisons, and subgroups

- Store follow-up magnitude and unit separately, such as `Followup_Value = 5`, `Followup_Unit = years`.
- Create separate rows for every outcome.
- Create separate rows for every comparison, including `A vs B`, `A vs C`, and `A vs D`.
- Create separate rows for every subgroup level, with fields such as `Subgroup_Variable = Sex`, `Subgroup_Level = Male`.
- Create separate rows for every adjusted model, statistical model, analysis population, or analysis role that changes the estimate.

## Data types and missingness

Numeric fields may contain an integer, decimal, or `NR`; use `UNCERTAIN` only for an unreadable candidate value. Do not embed `%`, years, months, `CI`, `HR=`, `n=`, or `p=` in numeric fields. Store units separately. Preserve P-value comparison symbols in `P_Value` as specified above.

Use only `NR` for not reported; never use blank, `NA`, `N/A`, `none`, or `not available`.

## No-calculation boundary

Default mode is `EXTRACTION ONLY`. Without explicit permission—such as the user stating `ALLOW CALCULATION`—do not calculate event rates, incidence, RR, OR, HR, IRR, SE, SD, person-years, N, CI, percentage-derived events, curve-derived values, or any other missing statistic.

Selecting a directly reported treatment-specific N that matches the Target PICO is extraction, not calculation. The value must be printed in the supplied PDF; do not derive it by subtraction, percentages, residual counts, or assumptions.

If the user explicitly allows calculations:

1. Keep reported inputs unchanged.
2. Put calculated values in separate clearly labeled fields.
3. Record formula, assumptions, software/method, and calculation status.
4. Never use a calculated value to conceal a reported-source discrepancy.

## Source cross-validation and priority

Compare Abstract, Results text, main table, figure, and supplement. If values conflict, preserve all versions and flag `DISCREPANCY`.

Only when repeated values are consistent, use this final provenance priority:

1. Main outcome table
2. Supplementary table
3. Figure numeric label
4. Results text
5. Abstract

Priority never authorizes overwriting a conflicting value.

## Primary-analysis selection

Select a primary record only after retaining all candidate rows in `Full_Extraction_Dataset`.

First require that the actual `Effect_Exposure`, `Effect_Comparator`, population, and outcome represent the Target PICO. Normally the row must have `Eligible_For_Primary_Meta_Analysis = YES`. Never select `SUBGROUP_N_ONLY` or relabel a broader comparison because a target-subgroup N happens to be reported.

Use this decision order:

1. The article explicitly designates the outcome, comparison, time point, population, and model as primary.
2. The prespecified primary analysis in Methods matches the principal estimate in Results or the main outcome table.
3. If no analysis is explicitly designated, choose the estimate most clearly presented as the main adjusted analysis that matches the question PICO, and set `Primary_Selection_Confidence = Moderate` with a concise basis.
4. If two or more candidates remain equally plausible, do not guess. Mark selection `UNCERTAIN`, list candidates in `Notes`, and add `Manual verification required`.

Sensitivity, subgroup, exploratory, secondary, and alternate-comparator estimates cannot replace an explicitly designated primary analysis. Do not equate “most adjusted” with “primary” without article support.

## Final row-level quality control

Check every normalized row for:

- N mistaken for person-years or person-years mistaken for N
- Number at risk mistaken for events, baseline N, censoring, or person-years
- Raw events or person-years mislabeled as weighted, matched, or adjusted
- Crude estimate mislabeled as adjusted
- Multivariable estimate mislabeled as IPTW, PSM, IPCW, or another model
- Sensitivity or subgroup result mislabeled as primary or overall
- Mixed outcomes, comparisons, follow-up times, subgroups, or models
- Two numeric values left in one normalized cell
- Units embedded in numeric fields
- Unreported values inferred or calculated without permission
- Arm count inconsistent with the exact intervention/exposure definition or population stage
- Study-reported group N overwritten by a PICO-aligned subgroup N, or vice versa
- PICO-aligned subgroup N paired with an effect estimate calculated for a broader population
- `Effect_Exposure` or `Effect_Comparator` relabeled to match the Target PICO rather than the actual study analysis
- `Comparison_Alignment` or `Eligible_For_Primary_Meta_Analysis` inconsistent with the directly reported effect population and outcome data
- Comparator direction reversed between the original source and normalized effect
- Missing source type, source ID, or page for a directly extracted value
- Discrepancy overwritten rather than retained

Correct any identified problem before delivery. If correction cannot be made from the PDF, use `UNCERTAIN`, lower confidence, preserve the competing source content, and request manual verification.

## Final validation gate for the primary dataset

Before a row enters `Primary_Meta_Analysis_Dataset`, verify all ten conditions:

1. The study population corresponds to `Target_Population`.
2. `Effect_Exposure` corresponds to the Target intervention/exposure.
3. `Effect_Comparator` corresponds to the Target comparator.
4. `Outcome` corresponds to `Target_Outcome`.
5. The effect estimate belongs to those same exposure/comparator populations.
6. N, events, person-years, and effect estimates are not mixed across populations, follow-up times, or models.
7. `Analysis_Role` and `Analysis_Model` are correctly classified.
8. No unreported number was inferred or calculated without permission.
9. `Source_Type`, `Source_ID`, page, confidence, and notes provide adequate provenance.
10. Cross-validation and QC reveal no unresolved critical discrepancy.

If a critical condition fails, set `Eligible_For_Primary_Meta_Analysis = NO`. If the evidence cannot be reliably determined, set it to `UNCERTAIN` and add `Manual verification required`.
