# Ordered PDF Extraction Workflow

Execute all applicable stages in order. Define Stage 0 once from the user's question. For multiple PDFs, complete Stages 1–12 independently for each PDF, then perform Stage 13.

## Stage 0 — Target PICO Definition

Before study extraction, create the question-level and normalized Target PICO defined in `pico-and-population.md`:

`Target_Population | Target_Intervention_or_Exposure | Target_Comparator | Target_Outcomes | Target_Effect_Measure | Target_Followup | Target_Study_Design`

Do not infer sample sizes. Store multiple target outcomes separately for downstream row generation.

## Stage 1 — PDF Structure Map

Scan the entire main PDF and uploaded supplement before normalizing values. Inventory every `TEXT`, `TABLE`, `FIGURE`, `SUPPLEMENT_TABLE`, `SUPPLEMENT_FIGURE`, and `APPENDIX` source, including outcome-related Methods and Results sections.

Create the `PDF_Structure_Map` schema from `data-schema.md`. This stage is source inventory only; do not merge or normalize values.

Use the printed PDF page number when reliably visible and also retain the file page index in an extension field when they differ. Mark uncertain locators explicitly.

## Stage 2 — Study Characteristics

Extract directly reported bibliographic, design, population, eligibility, exposure/intervention, comparator, outcome, date, setting, and data-source characteristics. Use one field per variable and `NR` when absent. Build the arm map and `Population_Flow` described in `pico-and-population.md`.

## Stage 3 — PICO-to-Study Group Mapping

This stage is mandatory before normalized sample sizes or effect estimates are attached to a target comparison.

1. Preserve `PICO_Target_Intervention` and `PICO_Target_Comparator`.
2. Extract `Study_Reported_Exposure` and `Study_Reported_Comparator`.
3. Classify `Intervention_Match` and `Comparator_Match` as `EXACT`, `SUBGROUP`, `PARTIAL`, or `NO`.
4. Preserve both study-reported and directly reported PICO-aligned group counts.
5. Lock every effect estimate to the actual `Effect_Exposure` and `Effect_Comparator`.
6. Classify `Comparison_Alignment` and provisional `Eligible_For_Primary_Meta_Analysis` according to `pico-and-population.md`.

Do not pair a target subgroup N with a broader-group effect estimate, and do not rename a broader comparison to match the Target PICO.

## Stage 4 — Original Table Reconstruction

Reconstruct every review-relevant table independently before decomposition.

For each table record `Study_ID`, table number, page, original title, original column structure, row labels, cells, and footnotes. The first-pass reconstruction must reproduce the article's original row × column structure and wording.

Do not rename rows or columns, change treatment/control order, split combined cells, merge categories, or combine crude/adjusted, weighted/unweighted, matched/unmatched, or alternate models during reconstruction. Preserve raw strings such as `133/38273` and `0.74 (0.60–0.93)` here; split them only in normalized datasets.

If a table continues across pages, reconstruct it as one table while retaining page provenance for each segment. If spanning headers or OCR make the structure unreliable, preserve the visible content, mark `UNCERTAIN`, and request manual verification.

## Stage 5 — Figure Reconstruction

Process every review-relevant figure independently. Record:

- Figure number and title
- Figure type
- X-axis and Y-axis
- Intervention/exposure group and comparator group
- Outcome
- Follow-up period
- Analysis type/model
- Figure legend and directly printed values

Allowed `Figure_Type` values include `Kaplan-Meier`, `Cumulative incidence`, `Forest plot`, `Flowchart`, `Bar chart`, `Line chart`, `Scatter plot`, and `Other`.

Render and visually inspect figure pages when layout or image labels matter. OCR may assist but never outranks the visible page; lower confidence when OCR or image quality limits certainty.

### Kaplan–Meier and survival figures

Extract every directly printed number-at-risk entry as one `Group × Time_Value` row:

`Study_ID | Figure_ID | Outcome | Group | Time_Value | Time_Unit | Number_at_Risk`

Extract directly printed events, total, log-rank P, effect estimate, and CI into separate fields. Do not estimate survival probability or any other value from curve position unless the user explicitly authorizes digitization/calculation. Never treat number at risk as events, N, person-years, censoring, or incidence.

### Forest plots

Extract each printed forest-plot row independently with:

`Study_ID | Figure_ID | Analysis_Category | Subgroup_Variable | Subgroup_Level | Exposure_Events | Exposure_PersonYears | Comparator_Events | Comparator_PersonYears | Effect_Measure | Effect_Estimate | CI_Lower | CI_Upper | P_Value | P_Interaction`

Read only printed numbers. Do not estimate from squares, diamonds, CI bars, axis positions, or symbol size.

### Flowcharts

Use flowcharts to populate `Population_Flow`, preserving exclusion stages, arm assignment, follow-up loss, and analysis populations. Do not collapse sequential counts into a single intervention N.

## Stage 6 — Text Extraction

After table and figure reconstruction, read Methods, Results, legends, and footnotes to add directly reported information not already available in a table or figure:

- Study design, setting, country, database, and study dates
- Population and eligibility definitions
- Intervention/exposure and comparator definitions
- Outcome definitions and codes
- Index date, follow-up, censoring, and competing-risk rules
- Statistical model and analysis role
- Adjustment variables
- Propensity score, weighting, matching ratio/caliper, and IPCW methods
- ITT, modified ITT, per-protocol, or as-treated definitions

Text may fill missing contextual fields but must not silently replace or rewrite a table or figure number. Preserve differences for cross-validation.

## Stage 7 — Numerical Decomposition

Normalize reconstructed strings only after their meaning is established from headers, Methods, legends, or footnotes. Follow every rule in `normalization-and-qc.md`. Retain the original raw value and source locator in an audit field or reconstruction sheet.

## Stage 8 — Analysis Classification and Analytical Row Generation

Generate rows only after group mapping, raw extraction, normalization, effect-population identification, and analysis classification.

One row is:

`Study × Effect_Exposure × Effect_Comparator × Outcome × Follow-up × Analysis_Model × Subgroup`

This key represents the actual analysis performed by the study. Store Target PICO fields separately. Retain `Analysis_Role`, the most specific directly supported `Analysis_Model`, `Comparison_Alignment`, and eligibility status on every row.

## Stage 9 — Cross-validation

For each review-relevant variable, compare Abstract, Results text, main table, figure, and supplement using `Cross_Validation` from `data-schema.md`. Mark `YES`, `NO`, `PARTIAL`, or `NOT_APPLICABLE`.

When values differ, preserve each source value, set `Consistent = NO` or `PARTIAL`, add `DISCREPANCY`, and do not silently adjudicate.

## Stage 10 — Quality Control

Run the final row-level checks in `normalization-and-qc.md`. Explicitly verify target-population alignment, actual comparison alignment, denominator integrity, model/role integrity, effect-population integrity, source traceability, and the absence of unauthorized calculations. Correct errors before proceeding; unresolved critical uncertainty must remain `UNCERTAIN — Manual verification required`.

## Stage 11 — Full Extraction Dataset

Create one row for every distinct analytic unit. Retain review-relevant crude, unadjusted, multivariable, IPTW, PSM, IPCW, ITT, per-protocol, secondary, subgroup, sensitivity, and exploratory analyses as separately labeled rows. Preserve `Comparison_Alignment` and `Eligible_For_Primary_Meta_Analysis`, including `NO` and `UNCERTAIN` rows needed to show why an analysis was excluded from the primary dataset.

## Stage 12 — Primary Meta-analysis Dataset

For each `Study × Target_Outcome × Target_Comparison`, include only an effect estimate whose actual exposure/comparator population represents the Target PICO comparison, normally requiring `Eligible_For_Primary_Meta_Analysis = YES`. Then retain the study-designated primary analysis or the analysis explicitly selected by the user's protocol. Use the primary-selection and final-gate rules in `normalization-and-qc.md`. Never include a subgroup-N-only comparison or automatically equate “most adjusted” with “primary.”

## Stage 13 — Multiple-study Master Dataset

Only after every study completes Stages 1–12, vertically append study datasets using standardized column names. Add new columns for directly reported variables found in later studies; fill those fields with `NR` for studies that do not report them. Never remove a new variable or merge unlike variables merely to align schemas.

After the normalized master dataset is complete, build `Frequency_Ranked_Outcome_Dataset` using `outcome-column-ranking.md`. This is a presentation/export layer: preserve the normalized long-format full and primary datasets as the auditable source of truth and do not mix analysis populations while pivoting.
