---
name: databricks-lineage-explorer
description: Review-only assistant that traces a column through the Marketing Campaign Databricks project (Raw CSV -> Bronze -> Silver -> Quality Check -> Gold -> Dashboard dataset -> Dashboard chart). Use whenever the user asks "trace column X", "where does X come from", "what depends on X", "how is X calculated", "which charts use X", or "what happens if I change X" for the marketing_campaign pipeline, even if they do not say "lineage". Reads only the supplied notebook exports and dashboard SQL files. Never connects to Databricks and never runs SQL.
---

# Databricks Lineage Explorer (Marketing Campaign project)

Trace one column from the raw Kaggle file to the dashboard, using only the evidence files the user supplies. This skill is adapted from a synthetic lineage exercise. That exercise had a section on semantic-model (DAX) measures. **That section is intentionally removed here**, because this project has no semantic model and no DAX. Its dashboard datasets are plain SQL queries on Gold and Silver tables.

## Scope

Layers, in order:

```
Raw CSV (Volume) -> Bronze -> Silver -> Quality Check -> Gold -> Dashboard dataset (SQL) -> Dashboard chart
```

All project tables live in `marketing.default`.

## Evidence files

Ask the user for exports of these. Use only what is supplied.

| Layer | Expected evidence |
|---|---|
| Raw + Bronze | `01_Bronze` notebook export (.py or .md) |
| Silver | `02_Silver` notebook export |
| Quality check | `04_quality_check` notebook export |
| Gold | `03_gold` notebook export |
| Dashboard datasets | The SQL text of each dashboard dataset (Data tab) |
| Dashboard charts | A list of charts with their dataset and fields (a screenshot or typed list is fine) |

If a file is missing, say so and continue with the layers you can confirm.

## Workflow

1. Identify the **requested column** and the **target object**. If the target is not given, trace the column across all layers.
2. Search each evidence file for the column name and for any renamed or derived version of it.
3. Follow the column forward layer by layer. Record the object, column name, and evidence file for each step.
4. Classify every statement as **Confirmed** (visible in evidence), **Derived** (a reasonable conclusion from confirmed facts, labeled as such), or **Not found** (no evidence).
5. Write the answer using the output format below.

## Output format

### 1. Source details
```text
Requested Column:
Target Object:
Source File: marketing_campaign.csv (tab-separated, in a Unity Catalog Volume)
Bronze Table: marketing.default.bronze_campaign
Source Column:
```

### 2. Column lineage
Show one line per layer, and cite the evidence file for each step.

```text
Raw CSV: <column>
    -> Bronze: marketing.default.bronze_campaign.<column>
    -> Silver: marketing.default.silver_campaign.<column or derived column>
    -> Gold: marketing.default.gold_by_<x>.<column or aggregate>
    -> Dashboard dataset: <dataset name>.<field>
    -> Dashboard chart: <chart title> (<chart type>, axis or field)
```

Skip a layer only if the evidence shows the column does not reach it, and say so.

### 3. Transformations
For each layer, state exactly what changes the column (filter, mapping, `CASE`/`when`, aggregation, rename). Quote the relevant expression from the evidence. Say "direct copy, no transformation shown" when that is the case. Bronze is expected to be a direct load.

### 4. Dependencies
- **Table dependencies:** tables built from this column (for example Gold tables that group or aggregate it).
- **Dashboard dependencies:** dataset names and charts that use it.
- **Quality check dependency:** whether a check in `04_quality_check` tests this column.
- Do not treat a name-only match as a confirmed dependency. Confirm it in the SQL or notebook code.

### 5. Optional validation query
Provide one illustrative query for the user to run themselves. Mark it "illustrative only, not executed".

```sql
SELECT <column>, COUNT(*) AS rows_count
FROM marketing.default.silver_campaign
GROUP BY <column>;
```

### 6. Gaps
List what the evidence does not show, for example a missing notebook, a dashboard dataset whose SQL was not supplied, or a chart whose fields are unknown. Write `Not found in the supplied evidence` and do not guess.

## Output rules

1. Use only the supplied evidence. Do not invent objects, joins, keys, mappings, transformations, or dependencies.
2. Cite the evidence file for every lineage claim.
3. Separate **Confirmed**, **Derived**, and **Not found**.
4. Do not connect to Databricks, access the workspace, or execute SQL.
5. Do not include a "Semantic Measure Dependencies" section. This project has no semantic model.
6. If the dashboard uses a calculated field inside a chart (for example an aggregation such as `SUM(...)` or `AVG(...)` set in the chart panel), report it under Transformations, labeled as a dashboard-level aggregation.

## Project reference (starting points, verify before use)

This is background from the project description, not evidence. Always confirm it against the supplied files before stating it as fact.

| Column | Where to look first |
|---|---|
| `Marital_Status` | Silver: values grouped into `Partnered` and `Single`; invalid values filtered out. Quality check tests the allowed values. |
| `Income` | Silver: rows with null income dropped, outlier filter applied; used to create `IncomeGroup`. Quality check tests for nulls. |
| `Year_Birth` | Silver: used to calculate `Age` and `AgeGroup`. |
| `Dt_Customer` | Silver: converted to a date; used to create `JoinYear`. |
| `Mnt*` spend columns | Silver: summed into `TotalSpend`. Gold: product spend table. |
| `Response`, `AcceptedCmp1`..`AcceptedCmp5` | Gold: average multiplied by 100 for response and acceptance rates. Dashboard: response KPI, campaign chart, offer funnel. |
| `Num*Purchases` | Gold: channel purchases table. Dashboard: purchases by channel chart. |
| `Education` | Gold: education summary. Dashboard: response by education chart and segment scorecard. |

## Example request

```text
Trace the lineage of Marital_Status in marketing.default.silver_campaign.
```

A good answer traces the column from the raw CSV through Bronze (direct load) to Silver (mapping to `Partnered` / `Single` and removal of invalid values), then to the quality check that validates it, then to any Gold table or dashboard dataset that uses it, and ends with a Gaps section.
