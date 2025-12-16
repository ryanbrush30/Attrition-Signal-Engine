# Attrition Signal Engine  
**Production-Grade Attrition Risk Modeling with Signal Integrity and KPI Alignment**

Attrition Signal Engine is a time-aware, production-first workforce risk modeling system designed to **reliably identify, score, and aggregate attrition risk** while enforcing **data coverage, join integrity, and operational survivability**.

It is built to run end-to-end in real operating environments, producing **actionable employee-, business-unit-, and KPI-aligned outputs**, not one-off notebook results.

---

## What Attrition Signal Engine Solves

Most attrition models fail in production due to:
- Missing or partial feature files
- Silent join failures
- Misaligned time windows
- Outputs that are not operationally usable

Attrition Signal Engine addresses these failure modes directly by prioritizing **robust execution, explicit validation, and usable outputs**.

It answers questions leaders actually ask:
- Who is at elevated attrition risk right now?
- Where is risk concentrating across teams or business units?
- Which signals are associated with elevated risk?
- How does workforce risk align with key business KPIs?
- Can this pipeline run repeatedly without breaking?

---

## Core Design Principles

- Production survivability over academic purity  
- Time-aware feature and label alignment  
- Explicit coverage and join integrity checks  
- Operational outputs first  
- Extensible signals without pipeline redesign  
- Clear separation between risk signals and causal claims  

---

## Key Capabilities

### 1. Full-History Signals Construction
If required signals are missing, the system can construct them instead of failing.

- Builds employee-period signal layers aligned to the modeling timeline
- Supports compensation, performance, manager, and behavioral signals
- Establishes a stable contract for future feature expansion

### 2. Time-Aware Employee-Period Panel
- Employee data is modeled as an employee-period panel
- Labels are forward-looking (stay or leave within a defined horizon)
- Designed to reduce common leakage patterns

### 3. Coverage and Join Integrity Checks
The system explicitly validates:
- Date range alignment across sources
- Unique employee-period key match rates
- Non-null rates for critical signals after joins

These checks prevent silent data failures and improve trust in outputs.

### 4. Signal-Rich Attrition Risk Scoring
Supports signals such as:
- Compensation history
- Performance history
- Manager assignment
- Overtime and PTO usage
- Internal transfers and job changes

Additional signals can be introduced without redesigning the pipeline.

### 5. Aggregation and KPI Alignment
Employee-level attrition risk is aggregated to:
- Business-unit and period level
- Selected business KPIs for exposure analysis

This alignment:
- Surfaces where workforce risk and business outcomes co-move
- Supports prioritization and scenario discussion
- Does **not** claim causal attribution or revenue impact

---

## Outputs and Artifacts

All outputs are written as Excel files to support direct review by HR, Finance, and Operations stakeholders.

| File | Description |
|----|----|
| `panel_scored.xlsx` | Employee-period panel with attrition risk scores |
| `panel_scores_min.xlsx` | Minimal employee-period risk output for operational use |
| `bu_period_features.xlsx` | Aggregated business-unit period features derived from employee risk |
| `bu_period_scored.xlsx` | Business-unit period attrition risk scores |
| `bu_period_kpi_scored.xlsx` | Business-unit risk aligned to selected KPIs |
| `kpi_impact_output.xlsx` | KPI exposure signals associated with workforce risk |
| `kpi_impact_summary.xlsx` | Executive-level KPI risk summary |

---

## Typical Pipeline Flow

1. Load employee, compensation, performance, and KPI data  
2. Construct or validate employee-period panel  
3. Validate joins and signal coverage  
4. Train or load attrition risk model  
5. Score employee-period attrition risk  
6. Aggregate risk to business-unit and period level  
7. Align aggregated risk to selected KPIs  
8. Write operational and executive-ready artifacts  

---

## Example: Join Integrity Validation

```python
panel = pd.read_excel("panel_scored.xlsx")
comp = pd.read_excel("comp_history.xlsx")

panel_keys = panel[["employee_id","period_start"]].drop_duplicates()
comp_keys = comp[["employee_id","period_start"]].drop_duplicates()

match = panel_keys.merge(
    comp_keys,
    on=["employee_id","period_start"],
    how="left",
    indicator=True
)

match_rate = (match["_merge"] == "both").mean()
print(f"Compensation key match rate: {match_rate:.3f}")
