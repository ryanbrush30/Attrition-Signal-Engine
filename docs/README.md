# Attrition-Signal-Engine

**Production-Grade Attrition Risk Modeling with Full-History Signals and Join Integrity Checks**

Impact Recov is a time-aware, production-first attrition risk pipeline designed to **reliably score employee attrition risk** while enforcing **data coverage, join integrity, and operational survivability**.  
It is built to run end-to-end in real environments, not as a one-off notebook.

---

## What Impact Recov Solves

Most attrition models fail in production because of:
- Missing or partial feature files
- Silent join failures
- Misaligned time windows
- Outputs that are not operationally usable

Impact Recov addresses these failure modes directly.

It answers questions leaders actually ask:
- Who is at risk right now?
- Where is risk concentrating?
- Which signals are driving risk?
- Can this pipeline run repeatedly without breaking?

---

## Core Design Principles

- **Production survivability over academic purity**
- **Time-aware feature and label alignment**
- **Explicit coverage and join integrity checks**
- **Operational outputs first**
- **Extensible signals without pipeline redesign**

---

## Key Capabilities

### 1. Full-History Signals Construction
If the signals file is missing, Impact Recov **builds it automatically** instead of failing.

- Constructs an employee-period signals layer aligned to the panel timeline
- Supports behavior, manager, and employment movement signals
- Designed as a stable contract for future feature expansion

### 2. Time-Aware Employee-Period Panel
- Employee data is structured as an employee-period panel
- Labels are forward-looking (stay or leave in horizon)
- Guards against common leakage patterns

### 3. Coverage and Join Integrity Checks
Impact Recov explicitly validates:

- Date range alignment between panel and signals
- Unique employee-period key match rate
- Non-null rates for critical signals after join

These diagnostics are logged and can be written as artifacts.

### 4. Signal-Rich Attrition Risk Scoring
Supports signals such as:
- Manager ID
- Overtime hours
- PTO usage and availability
- Internal transfer flag
- Job change flag

The model is designed to absorb additional signals like:
- Manager turnover
- Pay change and compa change
- Promotion flags
- Performance trends

### 5. Operational Outputs
Produces minimal, clean outputs designed for:
- Dashboards
- Action lists
- Workflow triggers
- Executive review

---

## Outputs and Artifacts

| File | Description |
|----|----|
| `artifacts/employee_period_panel.csv` | Time-aware employee-period panel used for training and scoring |
| `data/employee_period_signals_full.csv` | Full-history signals layer aligned to the panel |
| `artifacts/panel_scored.csv` | Panel with features and attrition risk scores |
| `artifacts/panel_scores_min.csv` | Minimal scoring output for operational use |
| `artifacts/join_quality_report.csv` | Match rate and non-null diagnostics for key signals (recommended) |

---

## Typical Pipeline Flow

1. Load or build the employee-period panel  
2. Check for signals file  
   - If missing, build full-history signals  
3. Validate:
   - Date range coverage
   - Employee-period key match rate
   - Non-null rates for critical fields  
4. Train or load model  
5. Score attrition risk  
6. Write operational artifacts  

---

## Example: Join Integrity Checks

```python
panel = pd.read_csv("artifacts/employee_period_panel.csv")
signals = pd.read_csv("data/employee_period_signals_full.csv")

panel_keys = panel[["employee_id","period_start"]].drop_duplicates()
sig_keys = signals[["employee_id","period_start"]].drop_duplicates()

match = panel_keys.merge(
    sig_keys,
    on=["employee_id","period_start"],
    how="left",
    indicator=True
)

match_rate = (match["_merge"] == "both").mean()
print(f"Signals key match rate: {match_rate:.3f}")
