# Data Schemas and Deliverables

Use these schemas as the minimum. Additional directly reported variables extend to the right as new, clearly named columns. Keep source provenance on normalized records.

## Workbook sheets

Default workbook order:

1. `Question_PICO`
2. `Target_PICO`
3. `Study_PICO_Mapping`
4. `PDF_Structure_Map`
5. `Study_Characteristics`
6. `Arm_Map`
7. `Population_Flow`
8. One raw reconstruction sheet per relevant table and figure, using unique Excel-safe names
9. `Number_at_Risk`
10. `Forest_Plot_Data`
11. `Cross_Validation`
12. `Full_Extraction_Dataset`
13. `Primary_Meta_Analysis_Dataset`
14. `Multiple_Study_Master_Dataset` when multiple studies are supplied
15. `Column_Frequency_Map`
16. `Frequency_Ranked_Outcome_Dataset`
17. `Extraction_Log`

If a workbook's sheet limit or size becomes impractical, place reconstructions in a clearly indexed companion workbook without dropping content.

## PDF Structure Map

| Column | Rule |
|---|---|
| `Study_ID` | Unique study identifier |
| `Source_Type` | `TEXT`, `TABLE`, `FIGURE`, `SUPPLEMENT_TABLE`, `SUPPLEMENT_FIGURE`, or `APPENDIX` |
| `Source_ID` | Printed section, table, figure, or appendix identifier |
| `Page` | Printed page when reliable; otherwise file page index with note |
| `Original_Title` | Exact title or section heading |
| `Main_Content` | Concise content inventory, not an interpretation |
| `Relevant_to_Meta_analysis` | `YES`, `NO`, or `UNCERTAIN` |

## Target PICO and study-group mapping

`Target_PICO` contains:

`Target_Population | Target_Intervention_or_Exposure | Target_Comparator | Target_Outcomes | Target_Effect_Measure | Target_Followup | Target_Study_Design`

`Study_PICO_Mapping` contains at minimum:

`Study_ID | PICO_Target_Intervention | PICO_Target_Comparator | Study_Reported_Exposure | Study_Reported_Comparator | Intervention_Match | Comparator_Match | Study_Reported_Exposure_N | Study_Reported_Comparator_N | PICO_Aligned_Intervention_N | PICO_Aligned_Comparator_N | Effect_Exposure | Effect_Comparator | Comparison_Alignment | Eligible_For_Primary_Meta_Analysis | Source_Type | Source_ID | Page | Extraction_Confidence | Notes`

Controlled values:

- `Intervention_Match`, `Comparator_Match`: `EXACT`, `SUBGROUP`, `PARTIAL`, or `NO`.
- `Comparison_Alignment`: `EXACT`, `SUBGROUP_WITH_FULL_OUTCOME_DATA`, `SUBGROUP_N_ONLY`, `PARTIAL`, or `NOT_ELIGIBLE`.
- `Eligible_For_Primary_Meta_Analysis`: `YES`, `NO`, or `UNCERTAIN`.

## Study Characteristics

One study per row, at minimum:

`Study_ID | First_Author | Publication_Year | Journal | Country | Region | Study_Design | Data_Source | Recruitment_Start | Recruitment_End | Followup_Start | Followup_End | Study_Population | Disease | Inclusion_Criteria | Exclusion_Criteria | Minimum_Age | Maximum_Age | Index_Date_Definition | Study_Reported_Exposure | Study_Reported_Comparator | Primary_Outcome | Secondary_Outcomes`

Keep separate outcome rows in analytic datasets even though `Secondary_Outcomes` is an inventory field here.

## Cross-validation

`Study_ID | Outcome | Variable | Abstract_Value | Text_Value | Table_Value | Figure_Value | Supplement_Value | Consistent | Discrepancy_Flag | Notes`

`Consistent` is `YES`, `NO`, `PARTIAL`, or `NOT_APPLICABLE`. Use `Discrepancy_Flag = DISCREPANCY` whenever sources conflict.

## Full Extraction Dataset core columns

Use at least:

`Study_ID | First_Author | Publication_Year | Country | Study_Design | Data_Source | Population | PICO_Target_Intervention | PICO_Target_Comparator | Target_Outcome | Study_Reported_Exposure | Study_Reported_Comparator | Study_Reported_Exposure_N | Study_Reported_Comparator_N | Intervention_Match | Comparator_Match | PICO_Aligned_Intervention_N | PICO_Aligned_Comparator_N | Comparison_Alignment | Eligible_For_Primary_Meta_Analysis | Effect_Exposure | Effect_Comparator | Outcome | Outcome_Type | Analysis_Role | Analysis_Model | Followup_Value | Followup_Unit | Subgroup_Variable | Subgroup_Level | Total_N | Exposure_Unweighted_N | Comparator_Unweighted_N | Exposure_Weighted_N | Comparator_Weighted_N | Exposure_Matched_N | Comparator_Matched_N | Exposure_Analysis_N | Comparator_Analysis_N | Exposure_Raw_Events | Comparator_Raw_Events | Exposure_Weighted_Events | Comparator_Weighted_Events | Exposure_Matched_Events | Comparator_Matched_Events | Exposure_Adjusted_Events | Comparator_Adjusted_Events | Exposure_Raw_PersonYears | Comparator_Raw_PersonYears | Exposure_Weighted_PersonYears | Comparator_Weighted_PersonYears | Exposure_Matched_PersonYears | Comparator_Matched_PersonYears | Exposure_Adjusted_PersonYears | Comparator_Adjusted_PersonYears | Exposure_Incidence | Comparator_Incidence | Incidence_Unit | Effect_Measure | Effect_Estimate | CI_Level | CI_Lower | CI_Upper | Standard_Error | P_Value | P_Interaction | Adjusted | Adjustment_Method | Source_Type | Source_ID | Page | Raw_Reported_Value | Extraction_Confidence | Notes`

`Effect_Exposure` and `Effect_Comparator` always describe the actual groups used to calculate the effect estimate. Target PICO and PICO-aligned N fields never redefine these actual effect populations.

### Controlled values

`Analysis_Role` must be one of:

- `Primary`
- `Secondary`
- `Subgroup`
- `Sensitivity`
- `Exploratory`

Use the article's most precise model name in `Analysis_Model`. Standard values include:

- `Crude`
- `Unadjusted`
- `Age-adjusted`
- `Sex-adjusted`
- `Multivariable-adjusted`
- `IPTW`
- `PSM`
- `IPCW`
- `IPTW + IPCW`
- `Fine-Gray`
- `Cox proportional hazards`
- `Logistic regression`
- `Poisson regression`
- `Negative binomial`
- `Per-protocol`
- `Intention-to-treat`
- `Sensitivity analysis`
- `Subgroup analysis`
- `Other`

When design population and statistical method are both needed, add separate fields such as `Analysis_Population` and `Regression_Model` instead of combining meanings ambiguously.

`Adjusted` is `YES`, `NO`, or `UNCERTAIN`. Put propensity/matching/weighting strategy in `Adjustment_Method` and retain distinct adjusted models as separate rows.

`Extraction_Confidence` is `High`, `Moderate`, or `Low`:

- `High`: directly and clearly reported.
- `Moderate`: complex layout but reliably readable.
- `Low`: OCR, image, figure, or layout makes the value potentially unreliable; add `Manual verification required` to `Notes`.

## Adjustment variables

Create one field per explicitly listed covariate:

`Adjusted_for_1 | Adjusted_for_2 | Adjusted_for_3 | ...`

Preserve article order when practical. If the covariate list is exceptionally large, use a separate long-format `Adjustment_Variables` sheet with `Study_ID`, analytic-row key, sequence, and covariate; do not collapse the list into an ambiguous string unless the user requests that representation.

## Wide extension examples

Directly reported study-specific data may add:

`Mean_Age_Exposure | SD_Age_Exposure | Mean_Age_Comparator | SD_Age_Comparator | Female_Exposure_Count | Female_Exposure_Percent | Female_Comparator_Count | Female_Comparator_Percent | Median_Followup_Exposure | IQR_Lower_Followup_Exposure | IQR_Upper_Followup_Exposure`

Never combine count and percent, or center and spread, in one normalized cell.

For a required wide number-at-risk representation, create one numeric field per group and time point:

`Risk_Time_0_Exposure | Risk_Time_0_Comparator | Risk_Time_1_Exposure | Risk_Time_1_Comparator | ...`

Retain the long-format `Number_at_Risk` sheet as the auditable source.

## Primary Meta-analysis Dataset

Use the same columns as the full dataset so it can be vertically appended or compared. Add:

`Primary_Selection_Basis | Primary_Selection_Confidence`

One row represents one primary `Study × Exposure × Comparator × Outcome × Follow-up × Analysis_Model × Subgroup` unit. Overall primary analyses should use `Subgroup_Variable = NR` and `Subgroup_Level = NR`.

For this dataset, interpret Exposure and Comparator in the analytic key as `Effect_Exposure` and `Effect_Comparator`. Normally require `Eligible_For_Primary_Meta_Analysis = YES`; do not include `SUBGROUP_N_ONLY` rows.

## Frequency-ranked outcome/effect export

Use `outcome-column-ranking.md` to create `Column_Frequency_Map` and `Frequency_Ranked_Outcome_Dataset`. The latter is the human-facing wide Excel/CSV view with exactly the required first 15 display columns, followed by frequency-ranked dynamic outcome/effect blocks. It does not replace the normalized source datasets.

`Column_Frequency_Map` contains at minimum:

`Block_ID | Canonical_Block_Name | Member_Columns | Anchor_or_Complete_Block_Basis | Distinct_Study_Frequency | Included_Study_Count | Frequency_Tier | Analysis_Role_Priority | Adjustment_Priority | Main_or_Sensitivity_Priority | Overall_or_Subgroup_Priority | Source_Priority | Final_Block_Order | Notes`

## Extraction Log

Record:

`Study_ID | Stage | Status | Issue_Type | Description | Source_ID | Page | Manual_Action_Required`

Use it for unreadable pages, missing supplements mentioned by the article, conflicting group labels, uncertain denominators, unresolved primary-analysis selection, and other limitations.
