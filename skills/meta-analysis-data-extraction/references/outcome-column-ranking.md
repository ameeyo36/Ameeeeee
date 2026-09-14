# Outcome and Effect Column Frequency Ranking

Read this reference after all included studies have completed extraction, normalization, eligibility classification, and QC. It governs the final human-facing wide Excel/CSV view. It does not replace `Full_Extraction_Dataset`, `Primary_Meta_Analysis_Dataset`, or their long-format row grain.

## Core principle

After the fixed first 15 columns, retain every directly reported outcome/effect-related variable that is relevant to the Target PICO, correctly matched to its analysis population, and not a duplicate representation of an existing standardized variable.

Do not keep only the most common measure. Rank eligible logical variable blocks by the number of distinct included studies that report them. Uncommon and study-specific variables remain in the dataset and appear toward the right. Use `NR` for studies that do not report a retained variable.

## Fixed first 15 columns

`Frequency_Ranked_Outcome_Dataset` must begin with these exact display columns in this exact order; do not insert identifiers, provenance, or dynamic fields before them:

1. `Author`
2. `Year`
3. `Country`
4. `Study design`
5. `Data source`
6. `N (EXPOSURE)`
7. `N (CONTROL)`
8. `Mean Age (EXPOSURE)`
9. `Mean Age (CONTROL)`
10. `Female (%) EXPOSURE`
11. `Female (%) CONTROL`
12. `Follow-up (years)`
13. `Event Type`
14. `Event (EXPOSURE)`
15. `Event (CONTROL)`

Map these display fields only from directly reported, population-aligned normalized values:

- `Author`, `Year`, `Country`, `Study design`, and `Data source` map from study characteristics.
- `N (EXPOSURE)` and `N (CONTROL)` must refer to the actual effect populations represented by that wide row. If models in the row use different denominators and no single common N is directly supported, use `NR` in the fixed fields and retain the model-specific Ns in dynamic blocks.
- Mean age and female percentage must be directly reported for the corresponding groups. Do not calculate percentages or convert other summaries.
- `Follow-up (years)` may contain a value only when follow-up is directly reported in years or conversion is explicitly authorized. Preserve non-year values in the normalized source dataset rather than silently converting them.
- `Event Type` is the outcome for the wide row.
- Fixed event fields contain a single directly reported, population-aligned event count common to that row. If raw, weighted, matched, or model-specific event counts differ, use `NR` here and retain each count in its correctly labeled dynamic block.

Mandatory means the columns must exist; it does not authorize filling an unsupported value.

## Safe wide-row construction

Build the wide view from validated normalized rows. Use at least this pivot identity:

`Study_ID × Effect_Exposure × Effect_Comparator × Outcome/Event Type × Follow-up × Subgroup`

Analysis models may appear as separate model-labeled column blocks in the same wide row only when the fields remain explicitly labeled and attached to the correct effect population. If models use materially different analysis populations, follow-up definitions, outcomes, subgroups, or comparison directions, create separate wide rows or retain model-specific denominators inside their blocks; never create a false shared denominator.

Maintain a stable internal row key and links back to the normalized source rows. The fixed display columns do not replace provenance.

## Candidate dynamic variables

Collect all directly reported, Target-PICO-relevant outcome/effect variables across the included study set, including when present:

- Before-weighting N, crude events, and at-risk person-years
- Crude effect estimate and confidence interval
- After-weighting N, adjusted events, and at-risk person-years
- Adjusted effect estimate and confidence interval
- IPTW, PSM, IPCW, ITT, per-protocol, sensitivity, or subgroup effects
- RR, OR, HR, IRR, RD, MD, SMD, incidence, incidence rate, P value, and P for interaction
- Other directly reported outcome variables that pass the relevance and population-alignment checks

Do not discard a variable because only one study reports it. Do not include a duplicate alias after standardization.

## Standardize before counting

Standardize synonymous labels before computing frequency. Use a semantic key such as:

`Analysis_Role + Analysis_Model + Effect_Measure + Value_Component + Population_Stage + Unit`

For example, “hazard ratio” and “HR” can share `Effect_Measure = HR`. “Adjusted hazard ratio,” “multivariable HR,” and “adjusted HR” may share an adjusted-HR block only when the PDF confirms methodologically equivalent adjusted models. If the covariate set, weighting method, analysis population, or model differs materially, preserve separate blocks.

Record every standardization decision in `Column_Frequency_Map`; do not merge fields merely because their labels look similar.

## Count distinct studies, not rows

For each candidate block calculate:

`Study_Frequency = count of distinct Study_ID values that directly report the qualifying variable/block`

Define the included-study denominator from the review/protocol and the dataset being exported. A candidate report qualifies only when it is directly reported, relevant to the Target PICO/outcome, correctly matched to the analysis population, and eligible for that export. For a primary export, normally require `Eligible_For_Primary_Meta_Analysis = YES`.

A study reporting the same adjusted HR in 12 subgroup rows contributes one study, not 12. Missing `NR` values do not count as reporting. A clearly identified but unreadable value may be tracked as reported-but-`UNCERTAIN` in the map, but must not be treated as a reliable extracted value.

## Rank logical blocks

Rank logical blocks rather than scattering related columns.

### Effect block integrity

Keep an effect estimate and its CI together, for example:

1. `Adjusted HR`
2. `Adjusted HR lower 95% CI`
3. `Adjusted HR upper 95% CI`

CI member frequencies do not separate them from the estimate. Use the effect estimate as the block anchor unless the user's protocol specifies a complete-block basis.

### Model block integrity

Where possible keep the same analysis stage together, for example:

1. `Before-weighting N`
2. `Before-weighting Crude events`
3. `Before-weighting At-risk person-years`
4. `Crude HR`
5. `Crude HR lower 95% CI`
6. `Crude HR upper 95% CI`

Then keep the corresponding after-weighting or adjusted block together. Do not interleave unrelated models solely because individual member-column frequencies differ.

### Ranking basis

For each block, record whether frequency is based on:

- the distinct-study frequency of a designated core/anchor variable, such as the effect estimate; or
- the distinct-study frequency of the complete block when completeness is the intended criterion.

Apply the chosen basis consistently and record it in `Anchor_or_Complete_Block_Basis`.

## Frequency tiers and final order

After the fixed 15 columns, order blocks as follows:

1. `Tier 1 — Most Common`: block or blocks with the largest distinct-study frequency.
2. `Tier 2 — Second Most Common`: block or blocks with the next-highest distinct frequency.
3. `Tier 3 — Less Common`: remaining shared blocks reported by at least two studies, sorted from higher to lower frequency.
4. `Tier 4 — Study-specific / Non-common`: blocks reported by one study, placed last.

If only one study is included, all blocks are study-specific/Tier 4; order them with the tie-break rules below. Do not manufacture additional tiers.

## Tie-breaking

When blocks have equal distinct-study frequency, order them by:

1. Primary analysis before secondary analysis.
2. Adjusted before crude when otherwise equally common.
3. Main analysis before sensitivity analysis.
4. Overall result before subgroup result.
5. Table-based directly reported result before figure-only result.

If a tie still remains, use stable alphabetical order of `Canonical_Block_Name` and document this final deterministic tie-break. Never delete a tied block.

## Technical and provenance fields

The required first 15 columns and all ranked dynamic blocks come first. If the deliverable needs row keys, eligibility, source, confidence, or notes, append those administrative/provenance columns after all ranked blocks or keep them in the normalized audit sheets. Never insert them into the fixed first 15.

## Final validation

Before delivery verify:

- The first 15 display columns are present, exact, and in order.
- Every eligible directly reported outcome/effect variable is represented once in the standardized block map.
- Frequencies count distinct studies rather than rows.
- All effect estimates remain adjacent to their lower and upper CI fields.
- Model-stage variables remain together when they form a logical block.
- Ties follow the required priority order.
- Every one-study variable is retained at the right and missing study values are `NR`.
- No synonymous duplicate survived, and no materially different model was incorrectly merged.
- Fixed N/events and dynamic model-specific values refer to the correct effect populations.
- No unsupported calculation or unit conversion was introduced.
- Every wide value can be traced to its normalized row and PDF source.

Do not finalize the Excel/CSV until these checks pass or unresolved items are marked `UNCERTAIN — Manual verification required`.
