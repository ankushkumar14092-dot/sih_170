# TECHNICAL DESIGN DOCUMENT
# SIH26170 — AI-Driven Anomaly Detection in Component Burn-In & Screening
# Indian Space Research Organisation (ISRO)

---

> **DISCLAIMER**: This document describes a prototype assistive decision-support system developed for Smart India Hackathon (SIH). It does NOT constitute official ISRO documentation, certified screening procedures, or validated acceptance criteria. All AI outputs are advisory. Mandatory engineering safety limits always take precedence. All validation in this document uses synthetic/simulated data unless explicitly stated otherwise.

---

## TABLE OF CONTENTS

1. Executive Summary
2. Problem Statement
3. Existing Screening Process
4. Existing System Limitations
5. Proposed Solution
6. Objectives
7. Scope & Assumptions
8. Functional Requirements
9. Non-Functional Requirements
10. High-Level Architecture
11. Physical Test Environment
12. Data Ingestion Architecture
13. Data Architecture & Database Design
14. Data Processing Pipeline
15. Feature Engineering
16. AI/ML Architecture
17. Module A — Dynamic Anomaly Detection
18. Module B — Drift Prediction
19. Multi-Parameter Correlation Analysis
20. Hybrid Risk Fusion Engine
21. Decision Engine
22. Explainable AI
23. Component Health Profile
24. Backend Architecture & Services
25. REST API Design
26. Frontend / Engineering Dashboard
27. Real-Time Architecture
28. Batch Processing Architecture
29. MLOps Architecture
30. Model Evaluation Strategy
31. Dataset Strategy
32. Safety & Fail-Safe Design
33. Security Architecture
34. Observability & Monitoring
35. Deployment Architecture
36. Testing Strategy
37. Auditability
38. Report Generation
39. System Diagrams (Mermaid)
40. Technology Stack & Architectural Decisions
41. Implementation Roadmap
42. SIH Demo Flow
43. Jury-Level Technical Defense
44. Limitations & Future Scope
45. Conclusion

---

## 1. EXECUTIVE SUMMARY

High-reliability electronic components used in space missions undergo Burn-In and Environmental Stress Screening (ESS) to eliminate early-life failures before deployment. Traditional screening relies on fixed electrical threshold checks at predefined time checkpoints. This approach is necessary but insufficient: a component can remain within its absolute specification limits while exhibiting abnormal drift, unusual temporal behaviour, or lot-relative outlier characteristics that are statistically significant precursors to latent failure.

This document describes the complete system architecture for an AI-assisted screening decision-support system that augments — not replaces — traditional threshold-based screening. The system implements two primary AI modules:

- **Module A**: Dynamic Outlier Detection — identifies components that are abnormal relative to their lot population even when they remain within absolute datasheet limits.
- **Module B**: Time-Series Drift Prediction — uses early burn-in measurements to predict the 168-hour value and assess drift risk.

The system is designed as a **Hybrid Safety + AI Architecture** where deterministic hard safety rules always take precedence over AI outputs. The AI layer provides behavioural screening, population-relative analysis, and predictive drift assessment as an assistive layer for engineering review.

The architecture is safe, explainable, auditable, modular, scalable, fail-safe, and implementable within the SIH prototype timeline.

---

## 2. PROBLEM STATEMENT

**Problem ID**: SIH26170
**Organization**: Indian Space Research Organisation (ISRO)
**Domain**: High-Reliability Electronic Components, Burn-In Testing, ESS, AI/ML, Predictive Reliability Engineering

Space-grade electronic components must achieve extremely high reliability. Burn-In and Environmental Stress Screening are standard processes to precipitate and eliminate early-life (infant mortality) failures. However:

1. Traditional screening checks fixed electrical/parametric thresholds at predefined checkpoints (0h, 24h, 96h, 168h).
2. A component can **pass all threshold checks** while exhibiting:
   - Abnormal drift relative to its lot population
   - Unusual temporal behaviour (non-monotonic, accelerating)
   - Lot-relative outlier behaviour (statistically significant deviation)
   - Abnormal parameter correlations
   - Intermittent anomalies between checkpoints
   - Early degradation signatures
   - Latent defect characteristics

**Illustrative Example** (ASSUMPTION — not real ISRO data):

| Parameter | Lot Average | Component Value | Absolute Limit | Traditional Result | AI Result |
|---|---|---|---|---|---|
| Leakage Current | 10 µA | 45 µA | 50 µA | PASS | FLAG FOR INVESTIGATION |

The component passes the hard limit but is 4.5× the lot average — a statistically significant population outlier that warrants engineering review.

The system must answer **both**:
1. Does the component violate mandatory engineering limits? (deterministic)
2. Does the component behave abnormally compared with healthy components throughout the screening process? (AI-assisted)

---

## 3. EXISTING SCREENING PROCESS

**[ASSUMPTION — general industry practice, not ISRO-proprietary]**

Standard burn-in and ESS for high-reliability components typically involves:

1. **Pre-screening electrical test** — baseline parametric measurements at ambient conditions (0h).
2. **Burn-In chamber loading** — components placed in burn-in boards inside temperature/voltage-stressed chambers.
3. **Stress application** — elevated temperature (typically 125°C for MIL-grade), elevated voltage, for defined duration (commonly 168 hours per MIL-STD-883).
4. **Intermediate checkpoints** — parametric measurements at 24h, 96h, 168h.
5. **Post-burn-in test** — final parametric measurements.
6. **Accept/Reject decision** — based on fixed threshold comparison at each checkpoint.
7. **Documentation** — test records, lot traceability.

Parameters typically measured include: standby current (Icc), leakage current (Ileak), propagation delay (tpd), output voltage levels, input threshold voltages, and functional test results.

---

## 4. EXISTING SYSTEM LIMITATIONS

| Limitation | Impact |
|---|---|
| Fixed threshold checks only | Cannot detect within-limit population outliers |
| No temporal trend analysis | Gradual drift not detected until limit violation |
| No lot-relative comparison | Abnormal components in a bad lot may all pass |
| No multi-parameter correlation | Correlated anomalies across parameters missed |
| No predictive capability | Cannot flag components likely to fail before 168h |
| Manual data analysis | Engineer bandwidth limits depth of review |
| No uncertainty quantification | No confidence measure on screening decisions |
| No automated explainability | Engineer must manually investigate flagged components |
| Reactive, not predictive | Defects detected only after limit violation |
| No audit trail for AI-assisted decisions | Reproducibility limited |

---

## 5. PROPOSED SOLUTION

A **Hybrid Safety + AI Screening Architecture** that:

1. **Preserves all mandatory hard safety rules** — deterministic threshold checks always execute first and always take precedence.
2. **Adds a dynamic AI screening layer** — population-relative outlier detection using robust statistical and ML methods.
3. **Adds a predictive drift layer** — early-measurement-based prediction of 168h values and drift risk.
4. **Adds multi-parameter correlation analysis** — detects abnormal parameter combinations.
5. **Fuses all signals into a calibrated risk score** — with configurable weights.
6. **Generates engineer-readable explanations** — for every flagged component.
7. **Maintains a complete audit trail** — for every decision.
8. **Provides an engineering dashboard** — for real-time and batch screening visibility.

The system is explicitly designed as an **assistive tool**. Every AI output is advisory. The engineer makes the final disposition decision.

---

## 6. OBJECTIVES

1. Detect within-limit population outliers that traditional threshold checking misses.
2. Predict 168h parametric values from early measurements (0h, 24h).
3. Quantify drift risk with uncertainty bounds.
4. Detect abnormal multi-parameter correlations.
5. Generate engineer-readable explanations for every anomaly flag.
6. Maintain a complete, reproducible audit trail for every screening decision.
7. Provide a real-time engineering dashboard.
8. Support both real-time telemetry and batch historical analysis.
9. Implement fail-safe behaviour — AI failure must never result in a silent dangerous PASS.
10. Demonstrate clear differentiation between static limit checking and dynamic behavioural screening.

---

## 7. SCOPE & ASSUMPTIONS

### In Scope
- Burn-In and ESS telemetry ingestion (real-time and batch)
- Dynamic anomaly detection (Module A)
- Drift prediction (Module B)
- Multi-parameter correlation analysis
- Risk fusion and decision engine
- Explainability engine
- Engineering dashboard
- REST API backend
- MLOps pipeline
- Audit trail
- SIH prototype deployment

### Out of Scope
- Official ISRO acceptance criteria definition
- Certified screening procedure replacement
- Real ISRO hardware integration (prototype uses synthetic data)
- Radiation effects modelling
- Mechanical/thermal stress modelling beyond parametric correlation

### Assumptions
- **[ASSUMPTION]** Burn-in checkpoints are at 0h, 24h, 96h, 168h.
- **[ASSUMPTION]** Parameters include: Icc (standby current), Ileak (leakage current), tpd (propagation delay), Voh, Vol, Vih, Vil.
- **[ASSUMPTION]** Lot sizes are 20–200 components for statistical validity.
- **[ASSUMPTION]** Temperature during burn-in is monitored and recorded.
- **[ASSUMPTION]** Hard safety limits are defined by the component datasheet and/or applicable MIL standard.
- **[ASSUMPTION]** The system operates in a secure, air-gapped or controlled network environment.
- **[ASSUMPTION]** All thresholds and weights in the risk fusion engine require calibration on real validated data before operational use.

---

## 8. FUNCTIONAL REQUIREMENTS

### FR-01: Telemetry Ingestion
The system shall accept telemetry via REST API, MQTT, CSV upload, and JSON batch import.

### FR-02: Schema Validation
Every telemetry record shall be validated against a defined schema before entering the processing pipeline.

### FR-03: Data Quality Gate
Records failing quality checks shall be quarantined and not enter the ML inference pipeline.

### FR-04: Hard Safety Rule Engine
The system shall evaluate every measurement against defined absolute limits. A hard limit violation shall immediately produce a FAIL/ENGINEERING REVIEW decision regardless of AI output.

### FR-05: Dynamic Anomaly Detection (Module A)
The system shall compute an anomaly score for each component relative to its lot population using robust statistical and ML methods.

### FR-06: Drift Prediction (Module B)
The system shall predict the 168h parametric value from early measurements and compute predicted drift, drift rate, and safety margin.

### FR-07: Multi-Parameter Correlation Analysis
The system shall detect abnormal parameter combinations where individual parameters remain within limits.

### FR-08: Risk Fusion
The system shall combine hard safety status, anomaly score, drift risk, correlation anomaly, and data quality into a single calibrated risk score.

### FR-09: Decision Engine
The system shall produce a deterministic decision (NORMAL / REVIEW / HIGH-RISK REVIEW / FAIL) based on defined precedence rules.

### FR-10: Explainability
Every non-NORMAL decision shall be accompanied by an engineer-readable explanation citing specific evidence.

### FR-11: Component Health Profile
The system shall maintain a digital profile for every component including its full screening timeline.

### FR-12: Audit Trail
Every decision shall have a complete, immutable audit trail including input data references, feature values, model version, and decision rationale.

### FR-13: Engineering Dashboard
The system shall provide a real-time engineering dashboard showing active screening status, anomalies, and predictions.

### FR-14: Report Generation
The system shall generate automated screening reports in PDF/HTML format.

### FR-15: Fail-Safe Behaviour
AI service failure shall never result in a silent PASS. The system shall fall back to hard safety rules only and flag the record for manual review.

---

## 9. NON-FUNCTIONAL REQUIREMENTS

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | API response latency (single component) | < 500ms |
| NFR-02 | Real-time telemetry processing latency | < 2s end-to-end |
| NFR-03 | Batch processing throughput | > 1000 records/minute |
| NFR-04 | System availability | > 99.5% (prototype) |
| NFR-05 | False negative rate (anomaly detection) | Minimized; reported explicitly |
| NFR-06 | Audit trail retention | Minimum 7 years (configurable) |
| NFR-07 | Model inference reproducibility | Deterministic given same input and model version |
| NFR-08 | Dashboard load time | < 3s |
| NFR-09 | Security | RBAC, TLS, encrypted at rest |
| NFR-10 | Explainability | Every alert must have human-readable reason |

---

## 10. HIGH-LEVEL ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHYSICAL TEST ENVIRONMENT                        │
│  Component Under Test → Burn-In Chamber → DAQ → Test Controller    │
└─────────────────────────────┬───────────────────────────────────────┘
                              │ Telemetry (MQTT / CSV / JSON)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA INGESTION LAYER                             │
│  Ingestion API → Schema Validation → Data Quality Engine           │
│  → Time Synchronization → Unit Normalization → Storage             │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA PROCESSING LAYER                            │
│  Cleaning → Noise Handling → Missing Value Handling                │
│  → Normalization → Temporal Alignment → Feature Engineering        │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 HYBRID AI SCREENING ENGINE                          │
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │ HARD SAFETY      │    │ DYNAMIC ANOMALY  │                      │
│  │ RULE ENGINE      │    │ DETECTOR         │                      │
│  │ (Deterministic)  │    │ (Module A)       │                      │
│  └────────┬─────────┘    └────────┬─────────┘                      │
│           │                       │                                 │
│           │              ┌────────┴─────────┐                      │
│           │              │ DRIFT PREDICTION │                      │
│           │              │ ENGINE (Mod B)   │                      │
│           │              └────────┬─────────┘                      │
│           │                       │                                 │
│           │              ┌────────┴─────────┐                      │
│           │              │ CORRELATION      │                      │
│           │              │ ANALYZER         │                      │
│           │              └────────┬─────────┘                      │
│           │                       │                                 │
│           └───────────┬───────────┘                                 │
│                       ▼                                             │
│              ┌─────────────────┐                                    │
│              │ RISK FUSION     │                                    │
│              │ ENGINE          │                                    │
│              └────────┬────────┘                                    │
│                       ▼                                             │
│              ┌─────────────────┐                                    │
│              │ EXPLAINABILITY  │                                    │
│              │ ENGINE          │                                    │
│              └────────┬────────┘                                    │
│                       ▼                                             │
│              ┌─────────────────┐                                    │
│              │ HYBRID DECISION │                                    │
│              │ ENGINE          │                                    │
│              └────────┬────────┘                                    │
└───────────────────────┼─────────────────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       NORMAL        REVIEW        FAIL
      / PASS        / ANOMALY    / HIGH RISK
          │             │             │
          └─────────────┼─────────────┘
                        ▼
          ┌─────────────────────────┐
          │  ALERT / ENGINEER       │
          │  REVIEW                 │
          └────────────┬────────────┘
                       ▼
          ┌─────────────────────────┐
          │  ENGINEERING DASHBOARD  │
          └────────────┬────────────┘
                       ▼
          ┌─────────────────────────┐
          │  SCREENING REPORT       │
          └────────────┬────────────┘
                       ▼
          ┌─────────────────────────┐
          │  AUDIT TRAIL            │
          └─────────────────────────┘
```

**MANDATORY RULE**: If a hard safety limit is violated, the deterministic safety result takes precedence. The AI layer cannot override a FAIL produced by the safety rule engine.


---

## 11. PHYSICAL TEST ENVIRONMENT

### Hardware Components

| Component | Role |
|---|---|
| Component Under Test (CUT) | Electronic component being screened |
| Burn-In Board | PCB fixture holding multiple CUTs in parallel |
| Burn-In Chamber | Temperature-controlled oven applying thermal stress |
| Temperature Sensors (PT100/thermocouple) | Monitor chamber and component junction temperature |
| Precision Voltage Source | Apply defined stress voltage to CUTs |
| Current Measurement (SMU/shunt) | Measure Icc, Ileak per component |
| Propagation Delay Tester | Measure tpd at defined voltage/temperature |
| DAQ System | Digitize and timestamp all measurements |
| Test Controller (PC/PLC) | Orchestrate test sequence, trigger measurements |
| Environmental Sensors | Humidity, ambient temperature |
| Network Interface | Transmit telemetry to ingestion layer |

### Telemetry Record Schema

```json
{
  "component_id": "CUT-LOT42-007",
  "test_id": "TEST-2024-001",
  "lot_id": "LOT-2024-042",
  "parameter": "leakage_current",
  "value": 45.2,
  "unit": "uA",
  "timestamp": "2024-01-15T08:00:00.000Z",
  "test_stage": "24h",
  "temperature": 125.0,
  "voltage": 5.5,
  "operating_condition": "burn_in_stress",
  "sensor_id": "DAQ-CH-07",
  "data_quality_flag": "VALID"
}
```

### Measurement Timeline

```
t=0h   → Pre-burn-in baseline (ambient, nominal voltage)
t=24h  → First intermediate checkpoint (stressed conditions)
t=96h  → Second intermediate checkpoint
t=168h → Final post-burn-in measurement
```

---

## 12. DATA INGESTION ARCHITECTURE

### Ingestion Sources

| Source | Format | Mode |
|---|---|---|
| DAQ / Test Equipment | JSON over MQTT | Real-time |
| Laboratory Export | CSV | Batch |
| Historical Archive | Parquet | Batch |
| Manual Entry | JSON via REST API | On-demand |

### Ingestion Pipeline

```
DATA SOURCE
    │
    ▼
INGESTION API (FastAPI / MQTT subscriber)
    │
    ▼
SCHEMA VALIDATION
  - Required fields present, data types correct, enum values valid
    │
    ├── FAIL → Quarantine Store + Alert
    │
    ▼
DATA QUALITY ENGINE
  - Missing value detection
  - Duplicate record detection (component_id + parameter + timestamp)
  - Invalid timestamp detection
  - Impossible value detection (negative current, temperature > 200°C)
  - Stuck sensor detection (same value N consecutive readings)
  - Unit mismatch detection
    │
    ├── FAIL → Quarantine Store + Data Quality Alert + Manual Review Flag
    │
    ▼
TIME SYNCHRONIZATION → UNIT NORMALIZATION → STORAGE
```

### Data Quality Rules

| Rule | Action |
|---|---|
| Missing required field | Quarantine, alert |
| Duplicate record | Deduplicate, log |
| Invalid timestamp | Quarantine, alert |
| Value outside physical bounds | Quarantine, alert |
| Stuck sensor (>5 identical consecutive) | Flag, alert |
| >20% missing values for a component | Block ML inference, flag for manual review |

**Principle**: Bad-quality data must not enter the ML inference pipeline. The system must fail loudly, not silently.

---

## 13. DATA ARCHITECTURE & DATABASE DESIGN

### Storage Strategy

| Store | Technology | Purpose |
|---|---|---|
| Raw Telemetry | TimescaleDB hypertable | Time-series raw measurements |
| Clean Telemetry | TimescaleDB hypertable | Validated, normalized measurements |
| Feature Data | PostgreSQL | Engineered features per component per checkpoint |
| Predictions | PostgreSQL | Drift predictions and anomaly scores |
| Screening Decisions | PostgreSQL | Final decisions with full audit context |
| Model Versions | PostgreSQL + Object Storage | Model metadata + serialized artifacts |
| Audit Logs | PostgreSQL (append-only) | Immutable audit trail |
| Cache | Redis | Lot statistics, active session data |
| Reports | Object Storage | Generated PDF/HTML reports |

### Entity Relationship Design

```
lots
  lot_id (PK), lot_number, component_type, manufacturer,
  manufacture_date, quantity, created_at

components
  component_id (PK), lot_id (FK→lots), serial_number,
  component_type, manufacturer_part_number, created_at

tests
  test_id (PK), lot_id (FK→lots), test_type, start_time,
  end_time, chamber_id, stress_temperature, stress_voltage,
  status, created_by

telemetry_raw  [TimescaleDB hypertable, partitioned by timestamp]
  id (PK), component_id (FK), test_id (FK), lot_id (FK),
  parameter, value, unit, timestamp, test_stage,
  temperature, voltage, operating_condition, sensor_id,
  data_quality_flag, ingestion_timestamp

telemetry_clean  [TimescaleDB hypertable]
  id (PK), raw_id (FK→telemetry_raw), component_id (FK),
  test_id (FK), parameter, value_normalized, unit_normalized,
  timestamp, test_stage, temperature, voltage, quality_score

features
  feature_id (PK), component_id (FK), test_id (FK),
  parameter, checkpoint, raw_value, moving_avg_3pt,
  rolling_std, slope_0h_to_now, rate_of_change, acceleration,
  deviation_from_lot_mean, deviation_from_lot_median,
  robust_z_score, percentile_in_lot, temp_normalized_value,
  computed_at

anomalies
  anomaly_id (PK), component_id (FK), test_id (FK),
  parameter, checkpoint, anomaly_score, population_deviation,
  anomaly_confidence, anomaly_type, model_version_id (FK),
  detected_at, explanation_json

predictions
  prediction_id (PK), component_id (FK), test_id (FK),
  parameter, predicted_168h_value, predicted_drift,
  drift_rate, safety_margin, prediction_confidence,
  lower_bound_95, upper_bound_95, model_version_id (FK),
  predicted_at

screening_results
  result_id (PK), component_id (FK), test_id (FK),
  hard_safety_status, anomaly_score, drift_risk_score,
  correlation_score, data_quality_score, risk_score,
  severity, decision, reason_codes (JSON),
  model_version_id (FK), decided_at, reviewed_by,
  review_action, review_timestamp, review_notes

alerts
  alert_id (PK), component_id (FK), test_id (FK),
  alert_type, severity, message, acknowledged,
  acknowledged_by, acknowledged_at, created_at

model_versions
  version_id (PK), model_name, model_type, version_tag,
  mlflow_run_id, artifact_path, parameters_json,
  metrics_json, status, deployed_at, deployed_by, created_at

audit_logs  [append-only, no UPDATE/DELETE]
  log_id (PK), entity_type, entity_id, action, actor,
  input_reference_json, feature_values_json, model_version_id,
  prediction_json, anomaly_score, decision, explanation_json,
  timestamp, session_id, ip_address
```

### Key Indexes

```sql
CREATE INDEX ON telemetry_raw (component_id, timestamp DESC);
CREATE INDEX ON telemetry_raw (lot_id, parameter, test_stage);
CREATE INDEX ON anomalies (test_id, anomaly_score DESC);
CREATE INDEX ON screening_results (test_id, decision);
CREATE INDEX ON audit_logs (entity_id, timestamp DESC);
```

### Retention Strategy

| Store | Retention |
|---|---|
| Raw Telemetry | 2 years online, archive to cold storage |
| Screening Results | 7 years (regulatory) |
| Audit Logs | 7 years, append-only, immutable |
| Model Artifacts | Indefinite (model registry) |

---

## 14. DATA PROCESSING PIPELINE

```
RAW TELEMETRY
    │
    ▼
VALIDATION → CLEANING → NOISE HANDLING
    │
    ▼
TEMPORAL ALIGNMENT
  - Align measurements to checkpoint windows (±30min tolerance)
    │
    ▼
NORMALIZATION
  - Temperature normalization: value_norm = value / (1 + alpha*(T - T_ref))
  - Robust scaling (median/IQR) for ML input features
    │
    ▼
FEATURE ENGINEERING (see Section 15)
    │
    ▼
ML-READY DATASET
```

Missing value strategy:
- < 20% missing → linear interpolation for time-series
- > 20% missing → block ML inference, flag for manual review

---

## 15. FEATURE ENGINEERING

| Feature | Engineering Purpose |
|---|---|
| raw_value | Baseline measurement |
| moving_avg_3pt | Smooth trend across checkpoints |
| rolling_std | Variability / instability indicator |
| slope_0h_to_now | Overall drift rate |
| rate_of_change | Instantaneous drift between checkpoints |
| acceleration | Second derivative — accelerating degradation |
| deviation_from_lot_mean | Population deviation |
| deviation_from_lot_median | Robust population deviation |
| robust_z_score = (v - median) / (1.4826 * MAD) | Outlier score robust to lot outliers |
| percentile_in_lot | Relative position in lot distribution |
| temp_normalized_value | Remove temperature confound |
| param_correlation_score (Mahalanobis) | Multi-parameter anomaly |
| time_under_stress | Cumulative stress exposure |
| value_range_0h_to_now | Total parametric excursion |
| trend_direction | Monotonic vs oscillating behaviour |

Lot-level statistics (per lot, per parameter, cached in Redis):
mean, median, std, MAD, min, max, 5th/95th percentile, IQR


---

## 16. AI/ML ARCHITECTURE

The AI engine is a layered architecture. No single black-box model is used. Each layer has a defined purpose, interpretable output, and a defined failure mode.

```
FEATURE MATRIX (per component, per checkpoint)
    │
    ├──────────────────────────────────────────────┐
    ▼                                              ▼
MODULE A                                    MODULE B
Dynamic Anomaly Detection                   Drift Prediction
    │                                              │
    ▼                                              ▼
Layer 1: Robust Statistical                 Layer 1: Feature Preparation
  (Z-score, MAD, percentile)                  (0h, 24h values + features)
    │                                              │
    ▼                                              ▼
Layer 2: Multivariate Detection             Layer 2: Regression Model
  (Isolation Forest + Mahalanobis)            (XGBoost / Ridge)
    │                                              │
    ▼                                              ▼
Layer 3: Temporal Anomaly                   Layer 3: Uncertainty Estimation
  (slope, acceleration analysis)              (prediction intervals)
    │                                              │
    └──────────────────┬───────────────────────────┘
                       ▼
              CORRELATION ANALYZER
              (Mahalanobis distance in
               multi-parameter space)
                       │
                       ▼
              RISK FUSION ENGINE
                       │
                       ▼
              EXPLAINABILITY ENGINE (SHAP)
                       │
                       ▼
              HYBRID DECISION ENGINE
```

---

## 17. MODULE A — DYNAMIC ANOMALY DETECTION

### Model Selection Rationale

| Model | Pros | Cons | Selected? |
|---|---|---|---|
| Robust Z-score / MAD | Interpretable, fast, lot-relative | Univariate only | Yes — Layer 1 |
| Mahalanobis Distance | Multi-parameter, interpretable | Assumes Gaussian, needs covariance | Yes — Layer 2 |
| Isolation Forest | Unsupervised, handles high-dim, fast | Less interpretable | Yes — Layer 2 primary |
| Local Outlier Factor | Density-based, good for clusters | Slow on large datasets, transductive | Backup/evaluation |
| One-Class SVM | Works with small datasets | Sensitive to hyperparameters, slow | Evaluation only |
| Autoencoder | Captures complex patterns | Requires more data, less interpretable, overkill for tabular | Not selected for prototype |

**Final Production Candidate**: Isolation Forest (primary) + Robust Z-score (interpretable layer) + Mahalanobis Distance (multi-parameter).

**Justification**: Burn-in datasets are typically small (20–200 components per lot), unlabeled (no ground-truth defect labels in prototype), and require interpretability for engineering review. Isolation Forest is computationally efficient, works well on small tabular datasets, and produces anomaly scores. The robust Z-score layer provides the interpretable "this component is X sigma from the lot median" explanation. Mahalanobis distance captures multi-parameter correlations. Deep learning (Autoencoder) is not justified for this dataset size and interpretability requirement.

### Module A Output

```json
{
  "component_id": "CUT-LOT42-007",
  "parameter": "leakage_current",
  "checkpoint": "24h",
  "anomaly_score": 0.73,
  "population_deviation": 3.52,
  "anomaly_confidence": 0.81,
  "anomaly_type": "POPULATION_OUTLIER",
  "robust_z_score": 3.52,
  "percentile_in_lot": 98.5,
  "lot_median": 10.2,
  "lot_mad": 1.8,
  "observed_value": 45.2,
  "model_version": "isolation_forest_v1.2.0"
}
```

### Isolation Forest Configuration

```python
IsolationForest(
    n_estimators=100,
    contamination=0.05,   # [ASSUMPTION] 5% expected anomaly rate — calibrate on real data
    max_features=1.0,
    random_state=42,
    n_jobs=-1
)
```

**[ASSUMPTION]**: The contamination parameter (expected anomaly fraction) must be calibrated using validated historical data before operational use. The value 0.05 is a prototype default.

### Lot-Level Training

The anomaly detector is trained per lot (or per component type if lot size is insufficient). This ensures population-relative detection. A minimum lot size of 10 components is required for statistical validity. [ASSUMPTION]

---

## 18. MODULE B — DRIFT PREDICTION

### Model Selection Rationale

| Model | Pros | Cons | Selected? |
|---|---|---|---|
| Linear Regression | Interpretable, fast | Cannot capture non-linear drift | Baseline |
| Ridge Regression | Regularized, interpretable | Linear only | Yes — simple cases |
| XGBoost | Handles non-linearity, feature importance, fast | Less interpretable than linear | Yes — primary |
| LightGBM | Faster than XGBoost, similar performance | Similar interpretability trade-off | Evaluation |
| Random Forest | Robust, feature importance | Slower, less precise extrapolation | Evaluation |
| LSTM / Temporal NN | Captures complex temporal patterns | Requires much more data, less interpretable | Not justified for 4-checkpoint series |

**Final Production Candidate**: XGBoost (primary) with Ridge Regression as interpretable baseline.

**Justification**: With only 4 checkpoints (0h, 24h, 96h, 168h), temporal neural models (LSTM) are not justified — there are insufficient time steps per component. XGBoost handles non-linear drift patterns, provides feature importance (used in explainability), and is computationally efficient. Ridge Regression serves as an interpretable baseline and sanity check.

### Input Features for Drift Prediction

```
value_0h
value_24h
slope_0h_to_24h
rate_of_change_0h_to_24h
temperature_0h
temperature_24h
component_type_encoded
lot_mean_0h
lot_mean_24h
lot_std_0h
```

### Module B Output

```json
{
  "component_id": "CUT-LOT42-007",
  "parameter": "leakage_current",
  "value_0h": 11.2,
  "value_24h": 18.5,
  "predicted_168h_value": 38.7,
  "predicted_drift": 27.5,
  "drift_rate": 0.163,
  "safety_margin": 11.3,
  "absolute_limit": 50.0,
  "prediction_confidence": 0.78,
  "lower_bound_95": 32.1,
  "upper_bound_95": 45.3,
  "drift_risk": "ELEVATED",
  "model_version": "xgboost_drift_v1.1.0"
}
```

### Prediction Uncertainty

Prediction intervals are computed using quantile regression (XGBoost supports quantile loss). This provides 95% prediction intervals, not just point estimates. Uncertainty is reported explicitly and used in the risk fusion engine — low-confidence predictions increase the risk score conservatively.

### Safety Criterion

**[ASSUMPTION — PROTOTYPE DECISION]**: A drift is flagged as elevated risk if:
- predicted_168h_value > 0.8 × absolute_limit, OR
- drift_rate > calibrated_threshold (to be determined from validated data)

These thresholds are configurable and must be calibrated by the appropriate engineering authority before operational use.

---

## 19. MULTI-PARAMETER CORRELATION ANALYSIS

### Purpose

Detect abnormal parameter combinations where each individual parameter remains within its absolute limit but the combination is statistically unusual.

**Example** [ASSUMPTION — illustrative]:
- Icc = 4.8 mA (limit: 5.0 mA) → PASS individually
- Ileak = 48 µA (limit: 50 µA) → PASS individually
- tpd = 9.8 ns (limit: 10 ns) → PASS individually
- All three simultaneously near their limits → Mahalanobis distance flags as multivariate outlier

### Method

1. Compute parameter correlation matrix per lot at each checkpoint.
2. Compute Mahalanobis distance for each component in the multi-parameter space.
3. Flag components with Mahalanobis distance > calibrated threshold (chi-squared distribution, p < 0.01). [ASSUMPTION]
4. Use SHAP on the Isolation Forest model trained on multi-parameter feature vectors to identify which parameter combinations drive the anomaly.

### Parameters Analyzed

- standby_current (Icc)
- leakage_current (Ileak)
- propagation_delay (tpd)
- output_voltage_high (Voh)
- output_voltage_low (Vol)
- temperature (T)

---

## 20. HYBRID RISK FUSION ENGINE

### Architecture

The risk fusion engine combines signals from all analysis layers into a single calibrated risk score.

```
INPUTS:
  hard_safety_status    → binary (0 = PASS, 1 = FAIL)
  anomaly_score         → [0, 1] from Module A
  population_deviation  → robust Z-score
  drift_risk_score      → [0, 1] from Module B
  correlation_anomaly   → [0, 1] from Mahalanobis
  prediction_confidence → [0, 1] from Module B
  data_quality_score    → [0, 1] from ingestion pipeline

FUSION FORMULA [PROTOTYPE — weights require calibration]:

  risk_score = w1 * anomaly_score
             + w2 * normalize(population_deviation)
             + w3 * drift_risk_score
             + w4 * correlation_anomaly
             + w5 * (1 - prediction_confidence) * drift_risk_score
             + w6 * (1 - data_quality_score)

  Default weights [ASSUMPTION — calibrate on validated data]:
    w1 = 0.30  (anomaly score)
    w2 = 0.25  (population deviation)
    w3 = 0.25  (drift risk)
    w4 = 0.10  (correlation anomaly)
    w5 = 0.05  (uncertainty penalty)
    w6 = 0.05  (data quality penalty)

OVERRIDE RULE:
  IF hard_safety_status == FAIL:
    risk_score = 1.0
    decision = FAIL (deterministic, AI cannot override)
```

### Risk Score to Severity Mapping [ASSUMPTION — calibrate thresholds]

| Risk Score | Severity | Decision |
|---|---|---|
| 0.00 – 0.40 | LOW | NORMAL |
| 0.40 – 0.60 | MEDIUM | REVIEW |
| 0.60 – 0.85 | HIGH | HIGH-RISK REVIEW |
| 0.85 – 1.00 | CRITICAL | FAIL / ENGINEERING REVIEW |
| Hard limit violated | CRITICAL | FAIL (deterministic) |

### Reason Codes

The engine generates human-readable reason codes:

```json
{
  "reason_codes": [
    "POPULATION_OUTLIER_LEAKAGE_CURRENT_3.5_SIGMA",
    "ELEVATED_DRIFT_RATE_LEAKAGE_CURRENT",
    "PREDICTED_168H_NEAR_LIMIT_82_PERCENT",
    "MULTIVARIATE_CORRELATION_ANOMALY"
  ]
}
```

---

## 21. DECISION ENGINE

### Decision Precedence (Deterministic)

```python
def make_decision(hard_safety_status, risk_score, anomaly_score,
                  drift_risk_score, prediction_confidence, data_quality_score):

    # Rule 1: Hard safety limit — always takes precedence
    if hard_safety_status == "FAIL":
        return Decision(
            label="FAIL",
            severity="CRITICAL",
            reason="MANDATORY_SAFETY_LIMIT_VIOLATED",
            ai_overridable=False
        )

    # Rule 2: AI service unavailable — conservative fallback
    if ai_service_unavailable:
        return Decision(
            label="MANUAL_REVIEW_REQUIRED",
            severity="UNKNOWN",
            reason="AI_SERVICE_UNAVAILABLE_CONSERVATIVE_FALLBACK",
            ai_overridable=False
        )

    # Rule 3: Data quality insufficient for ML inference
    if data_quality_score < DATA_QUALITY_THRESHOLD:
        return Decision(
            label="MANUAL_REVIEW_REQUIRED",
            severity="UNKNOWN",
            reason="INSUFFICIENT_DATA_QUALITY_FOR_AI_INFERENCE"
        )

    # Rule 4: Severe dynamic anomaly
    if risk_score >= FAIL_THRESHOLD:  # default 0.85
        return Decision(label="HIGH_RISK_REVIEW", severity="CRITICAL")

    # Rule 5: Elevated drift risk
    if drift_risk_score >= DRIFT_THRESHOLD:  # default 0.70
        return Decision(label="EARLY_INVESTIGATION", severity="HIGH")

    # Rule 6: Moderate anomaly
    if risk_score >= REVIEW_THRESHOLD:  # default 0.40
        return Decision(label="REVIEW", severity="MEDIUM")

    # Rule 7: Normal
    return Decision(label="NORMAL", severity="LOW")
```

**[PROTOTYPE DECISION]**: Decision labels (NORMAL, REVIEW, HIGH-RISK REVIEW, FAIL) are prototype categories for engineering review workflow. They are NOT official ISRO acceptance criteria. Final component disposition is always made by the qualified engineer.


---

## 22. EXPLAINABLE AI

### Principle

Every non-NORMAL decision must answer: **WHY WAS THIS COMPONENT FLAGGED?**

The explanation must be engineer-readable, cite specific evidence, and never claim causality unless experimentally established.

### Explanation Generation Pipeline

```
SHAP VALUES (from Isolation Forest / XGBoost)
    +
FEATURE CONTRIBUTIONS (ranked by |SHAP value|)
    +
NATURAL LANGUAGE TEMPLATE
    +
TREND VISUALIZATION DATA
    +
LOT COMPARISON DATA
    =
ENGINEER-READABLE EXPLANATION
```

### Explanation Schema

```json
{
  "component_id": "CUT-LOT42-007",
  "lot_id": "LOT-2024-042",
  "parameter": "leakage_current",
  "observed_value_24h": 45.2,
  "expected_value_lot_median": 10.2,
  "deviation_sigma": 3.52,
  "historical_baseline": {
    "0h": 11.2,
    "24h": 45.2
  },
  "trend": "INCREASING",
  "drift_rate_uA_per_hour": 0.163,
  "anomaly_score": 0.73,
  "predicted_168h": 38.7,
  "prediction_confidence": 0.78,
  "absolute_limit": 50.0,
  "safety_margin_percent": 22.6,
  "contributing_features": [
    {"feature": "robust_z_score", "shap_value": 0.31, "rank": 1},
    {"feature": "slope_0h_to_24h", "shap_value": 0.24, "rank": 2},
    {"feature": "deviation_from_lot_median", "shap_value": 0.18, "rank": 3}
  ],
  "decision_reason": "POPULATION_OUTLIER_ELEVATED_DRIFT",
  "narrative": "Leakage current remained below the absolute limit (45.2 µA vs limit 50 µA) but deviated substantially from the lot median (10.2 µA), corresponding to a robust Z-score of 3.52. The parameter shows a continuous increasing trend from 11.2 µA at 0h to 45.2 µA at 24h. The predicted 168h value is 38.7 µA (95% CI: 32.1–45.3 µA), with a safety margin of 22.6% from the absolute limit. This component is flagged for engineering review based on its population-relative behaviour and drift trajectory.",
  "disclaimer": "This explanation is generated by an AI assistive system. It does not constitute a certified screening decision. Final disposition is the responsibility of the qualified engineer."
}
```

### SHAP Usage

SHAP (SHapley Additive exPlanations) is applied to:
- XGBoost drift prediction model: explains which input features (value_0h, slope, lot_mean) drove the predicted 168h value.
- Isolation Forest: SHAP TreeExplainer provides feature contributions to the anomaly score.

SHAP is used for feature contribution ranking, not for causal claims.

---

## 23. COMPONENT HEALTH PROFILE

```json
{
  "component_id": "CUT-LOT42-007",
  "component_type": "74HC00 Quad NAND Gate",
  "lot_id": "LOT-2024-042",
  "manufacturer": "<manufacturer_name>",
  "test_id": "TEST-2024-001",
  "burn_in_conditions": {
    "temperature_C": 125,
    "voltage_V": 5.5,
    "duration_hours": 168
  },
  "timeline": {
    "0h":   {"leakage_current_uA": 11.2, "standby_current_mA": 0.8, "prop_delay_ns": 7.2},
    "24h":  {"leakage_current_uA": 45.2, "standby_current_mA": 0.9, "prop_delay_ns": 7.4},
    "96h":  {"leakage_current_uA": null, "note": "pending"},
    "168h": {"leakage_current_uA": null, "note": "predicted: 38.7"}
  },
  "anomaly_score": 0.73,
  "drift_prediction": {
    "predicted_168h_leakage_uA": 38.7,
    "drift_rate_uA_per_hour": 0.163,
    "confidence": 0.78
  },
  "risk_score": 0.68,
  "decision": "HIGH_RISK_REVIEW",
  "model_version": "v1.2.0",
  "last_updated": "2024-01-15T10:30:00Z"
}
```

---

## 24. BACKEND ARCHITECTURE & SERVICES

### Technology: Python + FastAPI + Pydantic

### Service Definitions

**1. Ingestion Service**
- Purpose: Accept telemetry from all sources (MQTT, REST, CSV, batch)
- Input: Raw telemetry records
- Processing: Parse, validate schema, route to validation service
- Output: Validated records to queue; rejected records to quarantine
- Failure Mode: Queue overflow → backpressure, alert; MQTT disconnect → reconnect with exponential backoff

**2. Validation Service**
- Purpose: Data quality checks
- Input: Schema-valid telemetry records
- Processing: Quality rules engine (missing, duplicates, impossible values, stuck sensors)
- Output: VALID records to clean store; INVALID to quarantine with reason codes
- Failure Mode: Falls back to conservative quarantine of all records

**3. Preprocessing Service**
- Purpose: Cleaning, normalization, temporal alignment
- Input: Clean telemetry records
- Processing: Interpolation, temperature normalization, checkpoint alignment
- Output: Preprocessed records ready for feature engineering
- Failure Mode: Returns error; does not pass incomplete data forward

**4. Feature Service**
- Purpose: Compute all engineered features
- Input: Preprocessed records + lot statistics (from Redis cache)
- Processing: Compute feature matrix per component per checkpoint
- Output: Feature vectors stored in features table
- Failure Mode: Returns partial features with missing flags; blocks ML inference if critical features missing

**5. Anomaly Detection Service**
- Purpose: Run Module A — dynamic outlier detection
- Input: Feature vectors
- Processing: Robust Z-score + Isolation Forest + Mahalanobis distance
- Output: anomaly_score, population_deviation, anomaly_type, confidence
- Failure Mode: Returns ANOMALY_SERVICE_UNAVAILABLE; triggers conservative fallback in decision engine

**6. Drift Prediction Service**
- Purpose: Run Module B — 168h drift prediction
- Input: Feature vectors (0h, 24h values + features)
- Processing: XGBoost inference + quantile regression for intervals
- Output: predicted_168h, drift_rate, safety_margin, confidence, bounds
- Failure Mode: Returns PREDICTION_SERVICE_UNAVAILABLE; decision engine uses conservative path

**7. Correlation Service**
- Purpose: Multi-parameter correlation analysis
- Input: Feature vectors for all parameters of a component
- Processing: Mahalanobis distance in multi-parameter space
- Output: correlation_anomaly_score, flagged_parameter_combinations
- Failure Mode: Returns CORRELATION_SERVICE_UNAVAILABLE; risk fusion uses conservative default

**8. Risk Fusion Service**
- Purpose: Combine all signals into risk score
- Input: hard_safety_status, anomaly_score, drift_risk, correlation_score, data_quality, confidence
- Processing: Weighted fusion formula with override rules
- Output: risk_score, severity, reason_codes
- Failure Mode: If any input is unavailable, uses conservative (higher risk) default

**9. Explainability Service**
- Purpose: Generate engineer-readable explanations
- Input: Feature vectors, SHAP values, anomaly scores, predictions
- Processing: SHAP computation + narrative template rendering
- Output: Explanation JSON + narrative text
- Failure Mode: Returns minimal explanation with available data; flags as PARTIAL_EXPLANATION

**10. Decision Service**
- Purpose: Apply deterministic decision precedence rules
- Input: Risk score, hard safety status, service availability flags
- Processing: Decision tree (see Section 21)
- Output: Decision label, severity, reason
- Failure Mode: AI unavailable → MANUAL_REVIEW_REQUIRED (never silent PASS)

**11. Alert Service**
- Purpose: Generate and route alerts for non-NORMAL decisions
- Input: Decision output
- Processing: Create alert record, notify via configured channels (dashboard, email, webhook)
- Output: Alert record in database, notification sent
- Failure Mode: Log alert locally; retry notification

**12. Model Management Service**
- Purpose: Model versioning, deployment, rollback
- Input: MLflow model registry
- Processing: Load production model version, validate, serve
- Output: Active model version for inference services
- Failure Mode: Fall back to previous stable version; alert ML engineer

**13. Reporting Service**
- Purpose: Generate automated screening reports
- Input: Screening results, component profiles, audit trail
- Processing: Render PDF/HTML report using Jinja2 templates
- Output: Report file stored in object storage, link returned via API

---

## 25. REST API DESIGN

### Authentication

All endpoints require JWT Bearer token. Role-based access control (RBAC) enforced per endpoint.

### Endpoints

---

**POST /api/v1/telemetry**
- Purpose: Ingest a single telemetry record
- Auth: TEST_ENGINEER, ADMIN
- Request:
```json
{
  "component_id": "CUT-LOT42-007",
  "test_id": "TEST-2024-001",
  "lot_id": "LOT-2024-042",
  "parameter": "leakage_current",
  "value": 45.2,
  "unit": "uA",
  "timestamp": "2024-01-15T08:00:00Z",
  "test_stage": "24h",
  "temperature": 125.0,
  "voltage": 5.5
}
```
- Response 202:
```json
{"record_id": "uuid", "status": "ACCEPTED", "quality_flag": "VALID"}
```
- Response 422: Schema validation error
- Response 400: Data quality failure with reason

---

**POST /api/v1/screening/analyze**
- Purpose: Trigger full AI screening analysis for a component at a checkpoint
- Auth: QA_ENGINEER, ADMIN
- Request:
```json
{"component_id": "CUT-LOT42-007", "test_id": "TEST-2024-001", "checkpoint": "24h"}
```
- Response 200:
```json
{
  "result_id": "uuid",
  "component_id": "CUT-LOT42-007",
  "decision": "HIGH_RISK_REVIEW",
  "risk_score": 0.68,
  "anomaly_score": 0.73,
  "drift_risk_score": 0.61,
  "reason_codes": ["POPULATION_OUTLIER_LEAKAGE_CURRENT_3.5_SIGMA"],
  "explanation": {...},
  "model_version": "v1.2.0",
  "disclaimer": "AI assistive output only. Engineer review required."
}
```

---

**POST /api/v1/anomaly/detect**
- Purpose: Run anomaly detection only (Module A)
- Auth: QA_ENGINEER, ML_ENGINEER, ADMIN
- Request: `{"component_id": "...", "test_id": "...", "checkpoint": "24h"}`
- Response 200: Anomaly detection result JSON

---

**POST /api/v1/drift/predict**
- Purpose: Run drift prediction only (Module B)
- Auth: QA_ENGINEER, ML_ENGINEER, ADMIN
- Request: `{"component_id": "...", "test_id": "...", "parameter": "leakage_current"}`
- Response 200: Drift prediction result JSON

---

**GET /api/v1/components/{component_id}**
- Purpose: Get component health profile
- Auth: All authenticated roles
- Response 200: Component health profile JSON
- Response 404: Component not found

---

**GET /api/v1/components/{component_id}/timeline**
- Purpose: Get full measurement timeline for a component
- Auth: All authenticated roles
- Response 200: Timeline JSON with all checkpoints and parameters

---

**GET /api/v1/components/{component_id}/anomalies**
- Purpose: Get all anomaly records for a component
- Auth: All authenticated roles
- Response 200: Array of anomaly records

---

**GET /api/v1/screening/{test_id}/result**
- Purpose: Get screening results for all components in a test
- Auth: QA_ENGINEER, ADMIN
- Response 200: Array of screening results with summary statistics

---

**GET /api/v1/models**
- Purpose: List all model versions in registry
- Auth: ML_ENGINEER, ADMIN
- Response 200: Array of model version records

---

**GET /api/v1/health**
- Purpose: System health check
- Auth: None
- Response 200: `{"status": "healthy", "services": {...}, "model_version": "v1.2.0"}`
- Response 503: Degraded with service status details

---

**POST /api/v1/telemetry/batch**
- Purpose: Ingest batch telemetry (CSV/JSON)
- Auth: TEST_ENGINEER, ADMIN
- Request: multipart/form-data with file
- Response 202: `{"batch_id": "uuid", "records_accepted": 450, "records_rejected": 3}`

---

## 26. FRONTEND / ENGINEERING DASHBOARD

### Technology: Next.js + TypeScript + Tailwind CSS + ECharts

### Pages

**1. Mission / Screening Command Center**
- Active tests count, healthy/anomaly/high-risk component counts
- Real-time alert feed
- Lot-level status summary cards
- System health indicators (AI services, data pipeline)

**2. Live Burn-In Monitoring**
- Real-time parameter trend charts per component (ECharts line charts)
- Color-coded component grid (green/yellow/red by risk score)
- Live anomaly score updates
- Checkpoint progress indicator

**3. Component Explorer**
- Search/filter by component_id, lot_id, decision, parameter
- Component health profile card
- Full measurement timeline chart
- Anomaly score history
- Drift prediction chart with confidence intervals

**4. Lot Analytics**
- Lot-level parameter distribution (box plots, violin plots)
- Population outlier visualization (scatter plot with flagged components highlighted)
- Lot-to-lot comparison
- Pass/Review/Fail breakdown per lot

**5. Anomaly Investigation**
- Anomaly detail view with full explanation
- SHAP feature contribution bar chart
- Component vs lot baseline comparison chart
- Temporal trend with anomaly markers
- Engineer review action panel (Accept / Escalate / Dismiss with notes)

**6. Drift Prediction**
- Predicted 168h value with confidence interval visualization
- Drift trajectory chart (0h → 24h → predicted 168h)
- Safety margin indicator
- Lot comparison of drift rates

**7. Screening Reports**
- Generate/download PDF report per component or lot
- Report history with audit trail link

**8. Model Monitoring**
- Active model versions
- Inference latency metrics
- Data drift indicators
- Model performance metrics (on labeled validation set)
- Retraining trigger status

**9. Audit Logs**
- Searchable audit trail
- Filter by component, engineer, decision, date range
- Full decision trace view

### UI Design Principles
- Engineering clarity over decorative design
- Color coding: green (NORMAL), yellow (REVIEW), orange (HIGH-RISK), red (FAIL)
- Every AI output labeled with confidence score and disclaimer
- No AI output presented without uncertainty quantification


---

## 27. REAL-TIME ARCHITECTURE

### Messaging Technology Decision

| Technology | Pros | Cons | Use Case |
|---|---|---|---|
| MQTT | Lightweight, low latency, designed for IoT/DAQ, pub/sub | No persistence by default, limited replay | SIH prototype, DAQ integration |
| Kafka | Persistent, replay, high throughput, partitioned | Heavy infrastructure, overkill for prototype | Production scale |
| Redis Streams | Simple, fast, lightweight persistence | Limited ecosystem vs Kafka | Medium scale |

**Decision**: MQTT for SIH prototype (matches DAQ/test equipment ecosystem). Kafka for production-scale deployment. Architecture is designed to swap the messaging layer without changing upstream or downstream services.

### Real-Time Pipeline

```
DAQ / Test Equipment
    │ MQTT publish (topic: burnin/telemetry/{lot_id}/{component_id})
    ▼
MQTT Broker (Eclipse Mosquitto)
    │
    ▼
INGESTION SERVICE (MQTT subscriber)
    │ async processing
    ▼
VALIDATION SERVICE
    │
    ▼
PREPROCESSING SERVICE
    │
    ▼
FEATURE SERVICE
    │
    ▼
ML INFERENCE (Anomaly + Drift + Correlation)
    │
    ▼
RISK FUSION SERVICE
    │
    ▼
DECISION ENGINE
    │
    ▼
ALERT SERVICE → Dashboard WebSocket push
    │
    ▼
DATABASE WRITE (async)
```

Target end-to-end latency: < 2 seconds from DAQ measurement to dashboard update.

---

## 28. BATCH PROCESSING ARCHITECTURE

```
CSV / Parquet / JSON file upload
    │
    ▼
BATCH INGESTION API (POST /api/v1/telemetry/batch)
    │
    ▼
FILE VALIDATION (schema, format, encoding)
    │
    ▼
RECORD-LEVEL VALIDATION (same rules as real-time)
    │
    ▼
PREPROCESSING (vectorized, pandas-based)
    │
    ▼
FEATURE ENGINEERING (batch computation)
    │
    ▼
ML INFERENCE (batch predict)
    │
    ▼
RISK FUSION (batch)
    │
    ▼
DECISION ENGINE (batch)
    │
    ▼
DATABASE WRITE (bulk insert)
    │
    ▼
REPORT GENERATION
    │
    ▼
NOTIFICATION (batch complete alert)
```

Batch processing supports historical analysis of archived burn-in data. Results are stored identically to real-time results and are fully auditable.

---

## 29. MLOPS ARCHITECTURE

### Pipeline

```
DATA VERSIONING (DVC / versioned Parquet snapshots)
    │
    ▼
DATA VALIDATION (Great Expectations / custom schema checks)
    │
    ▼
FEATURE ENGINEERING (reproducible pipeline, versioned)
    │
    ▼
TRAINING (scikit-learn / XGBoost / MLflow autolog)
    │
    ▼
VALIDATION (hold-out set, cross-validation)
    │
    ▼
EVALUATION (metrics: Recall, F1, FNR, MAE, RMSE, R²)
    │
    ▼
MODEL REGISTRY (MLflow Model Registry)
    │
    ▼
APPROVAL (ML Engineer + QA Engineer sign-off)
    │
    ▼
DEPLOYMENT (model artifact loaded by inference services)
    │
    ▼
INFERENCE (production serving)
    │
    ▼
MONITORING (data drift, model drift, performance)
    │
    ▼
RETRAINING TRIGGER (drift detected / scheduled / manual)
```

### MLflow Integration

- Every training run logged: parameters, metrics, artifacts, data version
- Model versions tagged: STAGING → PRODUCTION → ARCHIVED
- Rollback: previous PRODUCTION version retained; one-command rollback
- Experiment tracking: compare runs across hyperparameter configurations

### Data Drift Monitoring

- Monitor input feature distributions in production vs training distribution
- Use Population Stability Index (PSI) and KL divergence
- Alert when PSI > 0.2 (significant drift) [ASSUMPTION — calibrate threshold]
- Trigger retraining workflow automatically or via manual approval

### Model Drift Monitoring

- Monitor anomaly score distribution over time
- Monitor prediction error on labeled validation samples (if available)
- Alert when performance degrades beyond defined threshold

---

## 30. MODEL EVALUATION STRATEGY

### Module A — Anomaly Detection

Since real labeled defect data is unavailable in prototype, evaluation uses:
1. Synthetic data with injected known anomalies (ground truth available)
2. Held-out synthetic normal components (should not be flagged)

| Metric | Target | Rationale |
|---|---|---|
| Recall (True Positive Rate) | Maximize | Missing a defective component is high-risk |
| False Negative Rate | Minimize and report explicitly | Critical safety metric |
| Precision | Report (accept lower precision for higher recall) | False positives cause engineer workload |
| F1 Score | Report | Balance metric |
| PR-AUC | Report | Better than ROC-AUC for imbalanced datasets |
| ROC-AUC | Report | Standard benchmark |

**[IMPORTANT]**: In safety-critical screening, false negatives (missed defects) are more dangerous than false positives (unnecessary reviews). The system is tuned to minimize false negatives at the cost of higher false positive rate. This trade-off must be explicitly communicated to engineers.

### Module B — Drift Prediction

| Metric | Target |
|---|---|
| MAE (Mean Absolute Error) | Minimize; report in physical units (µA, ns) |
| RMSE | Minimize |
| R² | Maximize (> 0.85 target) [ASSUMPTION] |
| Error distribution | Report; check for systematic bias |
| Prediction interval coverage | 95% CI should contain true value 95% of time |

### System Performance

| Metric | Target |
|---|---|
| Single-component inference latency | < 500ms |
| Batch throughput | > 1000 records/minute |
| API availability | > 99.5% |
| Dashboard load time | < 3s |

### Explainability Evaluation

- Consistency: Same input → same SHAP values (deterministic)
- Stability: Small input perturbation → small explanation change
- Engineer interpretability: Qualitative review by domain expert

---

## 31. DATASET STRATEGY

### [ASSUMPTION] Real ISRO burn-in telemetry is not publicly available.

### Development Data Strategy

**1. Public Datasets (REAL DATA — labeled as such)**
- NASA Prognostics Data Repository (bearing, battery degradation datasets) — used for methodology validation only, not burn-in specific
- SECOM semiconductor manufacturing dataset (UCI) — process monitoring, used for feature engineering validation
- These are clearly labeled as REAL PUBLIC DATA used for methodology validation, not burn-in validation.

**2. Synthetic Burn-In Telemetry (SYNTHETIC DATA — labeled as such)**

Generated using physics-informed simulation:

```python
# Illustrative synthetic generation logic
def generate_component_telemetry(component_type, anomaly_type=None):
    # Base degradation model: Arrhenius-inspired drift
    # value(t) = value_0 * (1 + k * exp(Ea/kT) * t)
    # Parameters sampled from realistic distributions [ASSUMPTION]

    anomaly_types = {
        "gradual_drift":      lambda t: base * (1 + 0.003 * t),
        "sudden_spike":       lambda t: base + spike_at(t, t_spike),
        "intermittent":       lambda t: base + intermittent_noise(t),
        "temp_dependent":     lambda t: base * temp_factor(T(t)),
        "stuck_sensor":       lambda t: base if t < t_stuck else constant,
        "correlated_anomaly": lambda t: multi_param_anomaly(t),
        "latent_degradation": lambda t: base * (1 + slow_drift(t)),
        "lot_variation":      lambda t: base * lot_factor + noise(t)
    }
```

**3. Anomaly Injection (SIMULATED DATA — labeled as such)**

Known anomalies injected into synthetic normal data:
- Gradual drift: linear increase beyond lot norm
- Sudden spike: step change at random checkpoint
- Intermittent: random high values at some checkpoints
- Temperature-dependent drift: drift correlated with temperature excursion
- Correlated anomaly: multiple parameters simultaneously near limits
- Latent degradation: slow drift that approaches limit by 168h

**Labeling Convention**:
- `data_source: "SYNTHETIC"` — generated data
- `data_source: "SIMULATED_ANOMALY"` — injected anomaly
- `data_source: "REAL_PUBLIC"` — public dataset
- `data_source: "REAL_ISRO"` — real ISRO data (not available in prototype)

**[CRITICAL DISCLAIMER]**: Synthetic validation results do NOT constitute real-world ISRO hardware validation. The system requires validation on real burn-in telemetry before operational deployment.

---

## 32. SAFETY & FAIL-SAFE DESIGN

### Failure Scenarios and Responses

| Failure | System Response |
|---|---|
| ML model unavailable | Decision = MANUAL_REVIEW_REQUIRED; hard safety rules still execute; alert ML engineer |
| Database unavailable | Queue telemetry locally; alert; do not process new screening decisions |
| Telemetry missing (checkpoint gap) | Flag component as INCOMPLETE; block ML inference; flag for manual review |
| Sensor failure (stuck sensor) | Quarantine affected records; alert test engineer; flag component |
| Low prediction confidence (< 0.5) | Report uncertainty explicitly; increase risk score conservatively |
| Corrupted data | Quarantine; alert; do not process |
| Network failure | Local buffer; retry with exponential backoff; alert |
| Model version mismatch | Reject inference; alert ML engineer; use last known good version |
| Data drift detected | Alert ML engineer; flag predictions as POTENTIALLY_DEGRADED; do not auto-retrain without approval |

### Mandatory Fail-Safe Principle

**AI failure must NEVER silently result in a dangerous PASS.**

Implementation:
- Default decision when AI is unavailable: MANUAL_REVIEW_REQUIRED (not PASS)
- Hard safety rules execute independently of AI services
- Every AI service has a health check; decision engine checks service availability before inference
- All failures are logged to audit trail with timestamp and reason

### Data Quality Gate

ML inference is blocked if:
- Data quality score < configurable threshold (default 0.7) [ASSUMPTION]
- > 20% of required features are missing
- Critical parameters (Icc, Ileak) are missing at the current checkpoint

---

## 33. SECURITY ARCHITECTURE

### Roles (RBAC)

| Role | Permissions |
|---|---|
| ADMIN | Full access, user management, model deployment |
| QA_ENGINEER | View all, trigger analysis, review decisions, generate reports |
| TEST_ENGINEER | Ingest telemetry, view own test results |
| ML_ENGINEER | Model management, training, evaluation, view all data |
| VIEWER | Read-only access to dashboard and reports |

### Security Controls

| Control | Implementation |
|---|---|
| Authentication | JWT Bearer tokens (HS256, 8h expiry) |
| Authorization | FastAPI dependency injection with role check |
| Transport Security | TLS 1.3 for all API communication |
| Encryption at Rest | PostgreSQL TDE / filesystem encryption |
| Secrets Management | Environment variables via .env; production: AWS Secrets Manager / HashiCorp Vault |
| API Security | Rate limiting (slowapi), input validation (Pydantic), SQL injection prevention (SQLAlchemy ORM) |
| Audit Logging | Every API call logged with actor, timestamp, IP, action |
| Network Segmentation | Backend, database, ML services in separate Docker networks |
| Least Privilege | Each service has minimum required database permissions |
| Secure Model Storage | Model artifacts in access-controlled object storage |
| CORS | Restricted to known frontend origins |


---

## 34. OBSERVABILITY & MONITORING

### Metrics (Prometheus)

| Metric | Type | Purpose |
|---|---|---|
| api_request_duration_seconds | Histogram | API latency per endpoint |
| api_request_total | Counter | Request count by status code |
| telemetry_ingestion_total | Counter | Records ingested / rejected |
| ml_inference_duration_seconds | Histogram | Inference latency per model |
| ml_inference_errors_total | Counter | Inference failures |
| anomaly_score_distribution | Histogram | Distribution of anomaly scores |
| data_drift_psi | Gauge | Population Stability Index per feature |
| active_components_total | Gauge | Components currently in screening |
| decisions_by_label_total | Counter | NORMAL/REVIEW/FAIL counts |
| system_cpu_usage | Gauge | CPU utilization |
| system_memory_usage | Gauge | Memory utilization |

### Grafana Dashboards

1. System Overview — API health, latency, throughput
2. Screening Operations — active tests, decision distribution, alert rate
3. ML Model Health — inference latency, error rate, data drift PSI
4. Data Pipeline — ingestion rate, rejection rate, quality scores

### OpenTelemetry

Distributed tracing across services: telemetry record → ingestion → validation → preprocessing → feature → inference → decision → alert. Full trace available per record for debugging.

### Alerting Rules (Prometheus Alertmanager)

- API error rate > 5% for 5 minutes → alert
- ML inference latency > 2s → alert
- Data drift PSI > 0.2 → alert ML engineer
- Database connection failure → alert
- Any FAIL decision → alert QA engineer

---

## 35. DEPLOYMENT ARCHITECTURE

### A. SIH DEMO DEPLOYMENT

Single-machine Docker Compose deployment:

```
┌─────────────────────────────────────────────────────┐
│                  Docker Compose Host                │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Next.js  │  │ FastAPI  │  │ Synthetic        │  │
│  │ Frontend │  │ Backend  │  │ Telemetry Gen    │  │
│  │ :3000    │  │ :8000    │  │ (demo profile)   │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │TimescaleDB│ │  Redis   │  │  MQTT Broker     │  │
│  │ :5432    │  │ :6379    │  │  :1883           │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                                                     │
│  ┌──────────┐  ┌──────────┐                        │
│  │  MLflow  │  │ Grafana  │                        │
│  │ :5000    │  │ :3001    │                        │
│  └──────────┘  └──────────┘                        │
└─────────────────────────────────────────────────────┘
```

Start: `docker-compose --profile demo up --build`

### B. PRODUCTION-ORIENTED DEPLOYMENT

```
BURN-IN FACILITY (Physical)
    │ Secure network / VPN
    ▼
EDGE GATEWAY
  - Local MQTT broker
  - Edge validation (schema check)
  - Local buffer (network resilience)
  - Optional: edge ML inference for low-latency decisions
    │ Encrypted channel (TLS)
    ▼
TELEMETRY INGESTION CLUSTER
  - Load-balanced FastAPI ingestion services
  - Kafka for high-throughput message queue
    │
    ▼
AI SERVICES CLUSTER
  - Anomaly Detection Service (horizontally scalable)
  - Drift Prediction Service
  - Correlation Service
  - Risk Fusion Service
    │
    ▼
TIME-SERIES STORAGE (TimescaleDB cluster)
    │
    ▼
MODEL REGISTRY (MLflow + object storage)
    │
    ▼
ENGINEERING DASHBOARD (Next.js, CDN-served)
    │
    ▼
MONITORING (Prometheus + Grafana)
```

### Inference Modes

| Mode | Description | Use Case |
|---|---|---|
| Edge Inference | ML model runs on edge gateway | Ultra-low latency, network-resilient |
| Cloud/Server Inference | ML model runs on central server | Full feature set, model management |
| Hybrid | Hard safety rules at edge; full AI on server | Balance latency and capability |

**Recommendation**: Hybrid mode for production. Hard safety rules execute at edge (< 100ms). Full AI screening on central server (< 2s). Edge result is preliminary; server result is authoritative.

---

## 36. TESTING STRATEGY

### Test Scenarios

| Scenario | Input | Expected Decision | Engineer Action |
|---|---|---|---|
| 1. Normal component | All parameters within limits, near lot median | NORMAL | No action required |
| 2. Hard limit violation | Icc = 5.2 mA, limit = 5.0 mA | FAIL (deterministic) | Reject component |
| 3. Within-limit population outlier | Ileak = 45 µA, limit = 50 µA, lot median = 10 µA | HIGH-RISK REVIEW | Engineer investigates |
| 4. Gradual drift | Ileak: 10→18→32→(predicted 48) µA | HIGH-RISK REVIEW | Engineer investigates drift |
| 5. Sudden spike | tpd: 7→7→15→7 ns (spike at 96h) | REVIEW | Engineer checks 96h data |
| 6. Intermittent anomaly | Random high values at some checkpoints | REVIEW | Engineer checks pattern |
| 7. Multi-parameter anomaly | Icc, Ileak, tpd all near limits simultaneously | HIGH-RISK REVIEW | Multivariate flag |
| 8. Missing telemetry | 24h checkpoint data missing | MANUAL_REVIEW_REQUIRED | Engineer requests re-measurement |
| 9. AI service failure | Anomaly service returns 503 | MANUAL_REVIEW_REQUIRED (never PASS) | Engineer reviews manually |
| 10. Low-confidence prediction | Prediction confidence = 0.3 | REVIEW with uncertainty flag | Engineer notes low confidence |

### Test Types

- **Unit Tests**: Each service function tested in isolation with mocked dependencies
- **Integration Tests**: Service-to-service communication tested with test database
- **API Tests**: All endpoints tested with valid/invalid inputs, auth scenarios
- **ML Tests**: Model output range validation, determinism check, performance regression
- **Data Validation Tests**: Quality rules tested with crafted bad data
- **Model Regression Tests**: New model version must not degrade metrics beyond threshold
- **Load Tests**: Locust — 100 concurrent users, 1000 records/minute batch
- **Security Tests**: Auth bypass attempts, SQL injection, rate limiting
- **Failure Injection Tests**: Kill AI service → verify MANUAL_REVIEW_REQUIRED output
- **End-to-End Tests**: Full flow from telemetry ingestion to dashboard display

---

## 37. AUDITABILITY

Every screening decision has a complete, immutable trace:

```
TELEMETRY RECORD (raw_id reference)
    │
    ▼
DATA QUALITY RESULT (quality_score, flags)
    │
    ▼
SAFETY RULE RESULT (hard_safety_status, violated_limits)
    │
    ▼
ANOMALY DETECTION RESULT (anomaly_score, model_version)
    │
    ▼
DRIFT PREDICTION RESULT (predicted_168h, confidence, model_version)
    │
    ▼
CORRELATION ANALYSIS RESULT (correlation_score)
    │
    ▼
RISK FUSION RESULT (risk_score, weights_used, reason_codes)
    │
    ▼
EXPLANATION (narrative, SHAP values, feature contributions)
    │
    ▼
FINAL DECISION (label, severity, timestamp)
    │
    ▼
ENGINEER REVIEW (reviewer, action, notes, timestamp)
    │
    ▼
AUDIT LOG ENTRY (immutable, append-only)
```

Audit log entries cannot be modified or deleted. The system supports full decision replay: given the same input data and model version, the same decision is reproduced deterministically.

---

## 38. REPORT GENERATION

Automated screening reports contain:

1. Report header: test_id, lot_id, generated_at, generated_by, model_version
2. Lot summary: total components, NORMAL/REVIEW/FAIL counts, lot statistics
3. Per-component section:
   - Component details (ID, type, manufacturer)
   - Burn-in conditions (temperature, voltage, duration)
   - Parameter timeline table (0h, 24h, 96h, 168h values)
   - Parameter trend charts (matplotlib/ECharts)
   - Anomaly events with explanation
   - Drift prediction chart with confidence interval
   - Risk score and severity
   - Final decision
   - Engineer review record
4. Audit trail summary
5. Disclaimer: "This report is generated by an AI assistive system. All decisions require engineer review and approval. AI outputs are advisory only."
6. Model version and data source labels (SYNTHETIC / REAL)

---

## 39. SYSTEM DIAGRAMS (MERMAID)

### Diagram 1: High-Level System Architecture

```mermaid
graph TD
    CUT[Component Under Test] --> BIC[Burn-In Chamber]
    BIC --> DAQ[DAQ System]
    DAQ --> TGW[Telemetry Gateway / MQTT]
    TGW --> ING[Ingestion Service]
    ING --> VAL[Validation Service]
    VAL --> |VALID| PRE[Preprocessing Service]
    VAL --> |INVALID| QUA[Quarantine Store]
    PRE --> FEA[Feature Service]
    FEA --> HSR[Hard Safety Rule Engine]
    FEA --> MOD_A[Module A: Anomaly Detection]
    FEA --> MOD_B[Module B: Drift Prediction]
    FEA --> COR[Correlation Analyzer]
    HSR --> RFE[Risk Fusion Engine]
    MOD_A --> RFE
    MOD_B --> RFE
    COR --> RFE
    RFE --> EXP[Explainability Engine]
    EXP --> DEC[Decision Engine]
    DEC --> |NORMAL| PASS[PASS / NORMAL]
    DEC --> |REVIEW| REV[REVIEW]
    DEC --> |HIGH-RISK| HRR[HIGH-RISK REVIEW]
    DEC --> |FAIL| FAIL[FAIL]
    DEC --> ALS[Alert Service]
    ALS --> DASH[Engineering Dashboard]
    DEC --> AUD[Audit Trail]
    DEC --> RPT[Report Generation]
```

### Diagram 2: Physical Test Environment

```mermaid
graph LR
    subgraph Physical Hardware
        CUT1[CUT-001] & CUT2[CUT-002] & CUTN[CUT-N] --> BIB[Burn-In Board]
        BIB --> BIC[Burn-In Chamber]
        BIC --> TSENS[Temperature Sensors]
        BIC --> VSRC[Voltage Source]
        TSENS & VSRC --> DAQ[DAQ System]
        DAQ --> TC[Test Controller]
    end
    subgraph Software
        TC --> |MQTT/JSON| ING[Ingestion API]
    end
```

### Diagram 3: Data Flow

```mermaid
sequenceDiagram
    participant DAQ
    participant MQTT
    participant Ingestion
    participant Validation
    participant Features
    participant AnomalyDet
    participant DriftPred
    participant RiskFusion
    participant Decision
    participant Dashboard

    DAQ->>MQTT: publish telemetry record
    MQTT->>Ingestion: deliver record
    Ingestion->>Validation: validate schema + quality
    Validation-->>Ingestion: VALID / QUARANTINE
    Ingestion->>Features: compute feature vector
    Features->>AnomalyDet: feature matrix
    Features->>DriftPred: early values + features
    AnomalyDet-->>RiskFusion: anomaly_score
    DriftPred-->>RiskFusion: drift_risk_score
    RiskFusion->>Decision: risk_score + hard_safety_status
    Decision->>Dashboard: decision + explanation (WebSocket)
    Decision->>AuditLog: immutable audit entry
```

### Diagram 4: AI/ML Pipeline

```mermaid
graph TD
    FV[Feature Vector] --> L1A[Layer 1A: Robust Z-score / MAD]
    FV --> L1B[Layer 1B: Isolation Forest]
    FV --> L1C[Layer 1C: Mahalanobis Distance]
    L1A & L1B & L1C --> AS[Anomaly Score Aggregation]
    AS --> SHAP_A[SHAP Explainer - Module A]

    FV --> DP[XGBoost Drift Predictor]
    DP --> PI[Prediction Intervals - Quantile Regression]
    PI --> SHAP_B[SHAP Explainer - Module B]

    SHAP_A & SHAP_B --> EXP[Explainability Engine]
    AS --> RF[Risk Fusion]
    PI --> RF
    EXP --> RF
    RF --> DEC[Decision Engine]
```

### Diagram 5: Hybrid Decision Engine

```mermaid
flowchart TD
    START([Input: Component Measurement]) --> HSR{Hard Safety\nLimit Violated?}
    HSR --> |YES| FAIL([FAIL - Deterministic\nAI cannot override])
    HSR --> |NO| AI_OK{AI Services\nAvailable?}
    AI_OK --> |NO| MRR([MANUAL_REVIEW_REQUIRED\nConservative Fallback])
    AI_OK --> |YES| DQ{Data Quality\nSufficient?}
    DQ --> |NO| MRR
    DQ --> |YES| RS{Risk Score\n>= 0.85?}
    RS --> |YES| HRR([HIGH-RISK REVIEW])
    RS --> |NO| DR{Drift Risk\n>= 0.70?}
    DR --> |YES| EI([EARLY INVESTIGATION])
    DR --> |NO| MS{Risk Score\n>= 0.40?}
    MS --> |YES| REV([REVIEW])
    MS --> |NO| NRM([NORMAL])
```

### Diagram 6: Risk Fusion Engine

```mermaid
graph LR
    HSS[Hard Safety Status] --> RF[Risk Fusion Engine]
    AS[Anomaly Score w1=0.30] --> RF
    PD[Population Deviation w2=0.25] --> RF
    DR[Drift Risk Score w3=0.25] --> RF
    CA[Correlation Anomaly w4=0.10] --> RF
    UC[Uncertainty Penalty w5=0.05] --> RF
    DQ[Data Quality Penalty w6=0.05] --> RF
    RF --> RS[Risk Score 0-1]
    RF --> SEV[Severity Level]
    RF --> RC[Reason Codes]
    RS --> DEC[Decision Engine]
```

### Diagram 7: MLOps Lifecycle

```mermaid
graph TD
    DV[Data Versioning] --> DVL[Data Validation]
    DVL --> FE[Feature Engineering]
    FE --> TR[Model Training - MLflow autolog]
    TR --> VAL[Validation - hold-out set]
    VAL --> EVAL[Evaluation - Recall, FNR, MAE, R²]
    EVAL --> |Pass metrics| MR[Model Registry - MLflow]
    EVAL --> |Fail metrics| TR
    MR --> APR[Approval - ML Engineer + QA]
    APR --> DEP[Deployment - load artifact]
    DEP --> INF[Inference - production]
    INF --> MON[Monitoring - drift, performance]
    MON --> |Drift detected| RTR[Retraining Trigger]
    RTR --> DV
```

### Diagram 8: Database ER Diagram

```mermaid
erDiagram
    lots ||--o{ components : contains
    lots ||--o{ tests : has
    components ||--o{ telemetry_raw : generates
    tests ||--o{ telemetry_raw : records
    telemetry_raw ||--o| telemetry_clean : cleaned_to
    components ||--o{ features : has
    components ||--o{ anomalies : flagged_in
    components ||--o{ predictions : predicted_for
    components ||--o{ screening_results : screened_in
    model_versions ||--o{ anomalies : used_by
    model_versions ||--o{ predictions : used_by
    model_versions ||--o{ screening_results : used_by
    screening_results ||--o{ alerts : triggers
    screening_results ||--o{ audit_logs : logged_in
```

### Diagram 9: Deployment Architecture (SIH Demo)

```mermaid
graph TD
    subgraph Docker Compose
        FE[Next.js Frontend :3000]
        BE[FastAPI Backend :8000]
        DB[(TimescaleDB :5432)]
        RD[(Redis :6379)]
        MQ[MQTT Broker :1883]
        ML[MLflow :5000]
        GR[Grafana :3001]
        PR[Prometheus :9090]
        TG[Synthetic Telemetry Generator]
    end
    FE --> BE
    BE --> DB
    BE --> RD
    BE --> ML
    TG --> MQ
    MQ --> BE
    BE --> PR
    PR --> GR
```

### Diagram 10: Security Architecture

```mermaid
graph TD
    USER[User / Engineer] --> |HTTPS + JWT| API[FastAPI - Auth Middleware]
    API --> |RBAC check| SVC[Services]
    SVC --> |ORM - no raw SQL| DB[(PostgreSQL - TLS)]
    SVC --> |TLS| RD[(Redis)]
    SVC --> |TLS| ML[(MLflow)]
    API --> |append-only| AUD[(Audit Logs)]
    subgraph Network Segmentation
        API
        SVC
        DB
        RD
        ML
    end
```

### Diagram 11: Failure / Fallback Architecture

```mermaid
flowchart TD
    TEL[Telemetry Arrives] --> ING[Ingestion Service]
    ING --> |Schema invalid| QUA[Quarantine + Alert]
    ING --> |Valid| VAL[Validation]
    VAL --> |Quality fail| QUA2[Quarantine + Alert]
    VAL --> |Quality pass| AI[AI Inference]
    AI --> |Service available| FULL[Full AI Decision]
    AI --> |Service unavailable| SAFE[MANUAL_REVIEW_REQUIRED\nHard Safety Rules Only]
    FULL --> |Hard limit violated| FAIL[FAIL - Deterministic]
    FULL --> |AI result| DEC[AI-Assisted Decision]
    SAFE --> AUD[Audit Log - AI_UNAVAILABLE flag]
    FAIL --> AUD
    DEC --> AUD
```

### Diagram 12: Component Health Timeline

```mermaid
gantt
    title Component CUT-LOT42-007 Burn-In Timeline
    dateFormat HH
    axisFormat %Hh

    section Measurements
    Baseline 0h measurement     :milestone, m1, 00, 0h
    24h checkpoint              :milestone, m2, 24, 0h
    96h checkpoint              :milestone, m3, 96, 0h
    168h checkpoint             :milestone, m4, 168, 0h

    section AI Analysis
    Anomaly detection at 24h    :crit, a1, 24, 1h
    Drift prediction at 24h     :crit, a2, 24, 1h
    Alert generated             :crit, a3, 25, 1h

    section Engineer Review
    Engineer review             :active, r1, 26, 4h
```


---

## 40. TECHNOLOGY STACK & ARCHITECTURAL DECISIONS

### Technology Choices

| Layer | Technology | Justification | Alternative | Why Not Selected |
|---|---|---|---|---|
| Frontend | Next.js + TypeScript | SSR, type safety, React ecosystem | Vue.js, Angular | React ecosystem more mature for dashboards |
| UI Styling | Tailwind CSS | Utility-first, fast prototyping | Material UI | More flexible, less opinionated |
| Charts | ECharts / Recharts | Rich time-series support, performant | D3.js, Plotly | Lower-level D3 requires more code |
| Backend | FastAPI | Async, auto OpenAPI docs, Pydantic validation, Python ML ecosystem | Flask, Django | Flask lacks async; Django too heavy |
| Data Validation | Pydantic v2 | Fast, type-safe, integrates with FastAPI | Marshmallow | Pydantic v2 is faster and more Pythonic |
| Anomaly Detection | Isolation Forest + Robust Z-score | Unsupervised, small datasets, interpretable, fast | Autoencoder | Autoencoder needs more data, less interpretable |
| Drift Prediction | XGBoost + Ridge baseline | Non-linear drift, feature importance, fast | LSTM | Only 4 checkpoints — LSTM not justified |
| Explainability | SHAP | Model-agnostic, well-established, works with XGBoost/IF | LIME | SHAP more consistent, better for tree models |
| Primary Database | PostgreSQL + TimescaleDB | Relational integrity + time-series performance | MongoDB | Relational model better for audit trail and FK constraints |
| Time-Series | TimescaleDB | PostgreSQL extension, SQL interface, compression | InfluxDB | TimescaleDB keeps SQL interface, easier integration |
| Cache | Redis | Fast lot statistics cache, pub/sub | Memcached | Redis has richer data structures |
| Messaging | MQTT (prototype) / Kafka (production) | MQTT matches DAQ ecosystem; Kafka for scale | REST polling | Push model more efficient than polling |
| MLOps | MLflow | Open source, experiment tracking, model registry | Weights & Biases | MLflow self-hosted, no external dependency |
| Containerization | Docker + Docker Compose | Standard, reproducible, easy demo | Kubernetes | K8s overkill for SIH prototype |
| Monitoring | Prometheus + Grafana | Industry standard, open source | Datadog | Self-hosted, no cost |
| Tracing | OpenTelemetry | Vendor-neutral standard | Jaeger standalone | OTel is the standard |

### Key Architectural Decisions

**Decision 1: Why Isolation Forest over Autoencoder?**
- Dataset: Small (20–200 components per lot), unlabeled
- Isolation Forest: Works well on small tabular datasets, unsupervised, produces interpretable anomaly scores, computationally efficient
- Autoencoder: Requires larger datasets for meaningful reconstruction, less interpretable, adds PyTorch dependency without proportional benefit
- Trade-off: Isolation Forest may miss complex non-linear anomaly patterns that an Autoencoder could capture — acceptable for prototype, revisit with larger real datasets

**Decision 2: Why XGBoost over LSTM for drift prediction?**
- Data: 4 checkpoints per component (0h, 24h, 96h, 168h)
- XGBoost: Handles non-linear relationships, provides feature importance (used in SHAP), fast inference, works well with small datasets
- LSTM: Requires many time steps per sequence for meaningful temporal learning; 4 checkpoints is insufficient; adds complexity without benefit
- Trade-off: XGBoost cannot capture complex temporal dependencies — acceptable given 4-checkpoint constraint

**Decision 3: Why FastAPI over Flask?**
- FastAPI: Native async support (critical for real-time telemetry), automatic OpenAPI documentation, Pydantic integration, better performance
- Flask: Synchronous by default, no built-in validation, manual documentation
- Trade-off: FastAPI has steeper learning curve — acceptable given team Python expertise

**Decision 4: Why PostgreSQL/TimescaleDB over MongoDB?**
- PostgreSQL: ACID compliance, foreign key constraints, audit trail integrity, SQL for complex analytical queries
- TimescaleDB: Time-series optimized queries, automatic partitioning, compression — all as a PostgreSQL extension
- MongoDB: Document model less suited for relational audit trail; no native time-series optimization
- Trade-off: PostgreSQL schema is less flexible than MongoDB — acceptable given well-defined telemetry schema

**Decision 5: Why MQTT over direct REST streaming?**
- MQTT: Designed for IoT/DAQ, lightweight, pub/sub decouples DAQ from backend, handles intermittent connectivity
- REST polling: Higher latency, more bandwidth, tighter coupling
- Trade-off: MQTT requires broker infrastructure — acceptable, Mosquitto is lightweight

---

## 41. IMPLEMENTATION ROADMAP

### Phase 1: Synthetic Data Generator (Week 1)
- Deliverables: Python script generating realistic burn-in telemetry with configurable anomaly injection
- Dependencies: NumPy, Pandas
- Expected Output: CSV/JSON files with labeled synthetic data (NORMAL, GRADUAL_DRIFT, SUDDEN_SPIKE, etc.)
- Testing: Verify statistical properties match expected distributions

### Phase 2: Data Pipeline (Week 1–2)
- Deliverables: Ingestion API, validation service, preprocessing service, TimescaleDB schema, migrations
- Dependencies: Phase 1 data
- Expected Output: Clean telemetry stored in database; quarantine working
- Testing: Data quality rules tested with crafted bad records

### Phase 3: Dynamic Anomaly Detection — Module A (Week 2)
- Deliverables: Feature engineering, Isolation Forest + Z-score + Mahalanobis implementation, MLflow experiment tracking
- Dependencies: Phase 2 pipeline
- Expected Output: Anomaly scores on synthetic data; evaluation metrics on labeled synthetic set
- Testing: Recall > 0.80 on injected anomalies [ASSUMPTION — target]

### Phase 4: 168h Drift Prediction — Module B (Week 2–3)
- Deliverables: XGBoost drift predictor, quantile regression intervals, Ridge baseline
- Dependencies: Phase 2 pipeline
- Expected Output: Predicted 168h values with confidence intervals; MAE, RMSE, R² metrics
- Testing: R² > 0.85 on synthetic validation set [ASSUMPTION — target]

### Phase 5: Risk Fusion Engine (Week 3)
- Deliverables: Configurable risk fusion service, decision engine, reason code generation
- Dependencies: Phases 3, 4
- Expected Output: Risk scores and decisions for all test scenarios
- Testing: All 10 test scenarios produce expected decisions

### Phase 6: Explainability Engine (Week 3)
- Deliverables: SHAP integration, narrative template engine, explanation JSON schema
- Dependencies: Phases 3, 4, 5
- Expected Output: Engineer-readable explanations for all non-NORMAL decisions
- Testing: Explanation consistency check; qualitative review

### Phase 7: Backend APIs (Week 3–4)
- Deliverables: All REST endpoints, authentication, RBAC, audit logging
- Dependencies: Phases 2–6
- Expected Output: Fully functional API with OpenAPI documentation
- Testing: API tests for all endpoints; auth bypass tests; load test

### Phase 8: Engineering Dashboard (Week 4)
- Deliverables: Next.js dashboard with all 9 pages, real-time WebSocket updates
- Dependencies: Phase 7 API
- Expected Output: Functional dashboard showing live screening status
- Testing: E2E tests; UI review with engineering clarity criteria

### Phase 9: MLOps Pipeline (Week 4–5)
- Deliverables: MLflow integration, model registry, retraining scripts, drift monitoring
- Dependencies: Phases 3, 4
- Expected Output: Reproducible training pipeline; model versioning working
- Testing: Model regression tests; rollback test

### Phase 10: Testing & SIH Demo (Week 5)
- Deliverables: Full test suite, demo script, Docker Compose demo deployment, documentation
- Dependencies: All phases
- Expected Output: SIH-ready demo; all test scenarios passing
- Testing: Full E2E demo run; jury Q&A preparation

---

## 42. SIH DEMO FLOW (2–3 minutes)

**Objective**: Clearly demonstrate the difference between static limit checking and AI-assisted behavioural screening.

**Step 1** (0:00–0:20): Upload/select a synthetic burn-in lot
- Show lot selection screen
- Display lot of 50 components loading

**Step 2** (0:20–0:40): Display normal population
- Show lot analytics page
- Box plot of leakage current distribution
- Most components clustered around 10 µA

**Step 3** (0:40–1:00): Show a component that passes hard limits
- Select component CUT-LOT42-007
- Show: Ileak = 45.2 µA, Absolute Limit = 50 µA
- Traditional result: **PASS** ✓
- Highlight: "Traditional threshold checking says PASS"

**Step 4** (1:00–1:20): AI identifies it as a lot-relative anomaly
- Show anomaly detection result
- Robust Z-score = 3.52 (3.5 sigma from lot median)
- Percentile in lot = 98.5%
- Anomaly score = 0.73
- AI result: **FLAG FOR INVESTIGATION**
- Highlight: "AI sees this component is 4.5× the lot median"

**Step 5** (1:20–1:40): Show temporal trend
- Show parameter timeline chart: 11.2 µA → 45.2 µA (0h to 24h)
- Continuous increasing trend
- Compare with lot median trend (flat ~10 µA)

**Step 6** (1:40–2:00): Predict 168h value
- Show drift prediction: predicted 168h = 38.7 µA
- 95% CI: 32.1–45.3 µA
- Safety margin: 22.6% from limit
- Drift rate: 0.163 µA/hour

**Step 7** (2:00–2:15): Show drift risk
- Risk score: 0.68 → HIGH-RISK REVIEW
- Reason codes displayed

**Step 8** (2:15–2:30): Generate explanation
- Show full explanation narrative
- SHAP feature contribution chart
- "Leakage current remained below the absolute limit but deviated substantially from the lot median..."

**Step 9** (2:30–2:45): Show final engineering recommendation
- Decision: HIGH-RISK REVIEW
- Disclaimer shown prominently
- Engineer review action panel

**Step 10** (2:45–3:00): Open audit trail
- Show complete decision trace
- Every step logged with timestamp and model version
- Demonstrate reproducibility

**Key Message**: "The component passed the traditional threshold check. The AI system identified it as a population outlier with an accelerating drift trajectory. This is the difference between static limit checking and dynamic behavioural screening."

---

## 43. JURY-LEVEL TECHNICAL DEFENSE

**Q: Why is AI required? Can't engineers just look at the data?**
A: Engineers can review data manually, but a typical burn-in lot has 50–200 components, each with 5–7 parameters measured at 4 checkpoints — that is 1,000–5,600 data points per lot. Manual review of all parameter trends and population comparisons is impractical at scale. The AI system automates population-relative comparison and drift trend analysis, flagging the specific components that warrant engineer attention. The engineer's expertise is then focused where it matters most.

**Q: Why are static thresholds insufficient?**
A: Static thresholds check whether a value exceeds a fixed limit. They cannot detect: (1) a component that is 4.5× the lot average but still below the limit; (2) a component with an accelerating drift trajectory that will approach the limit after deployment; (3) abnormal combinations of parameters where each is individually within limits. These are exactly the patterns associated with latent defects and infant mortality failures.

**Q: How do you detect latent defects?**
A: We do not claim guaranteed detection of latent defects — that would be scientifically indefensible. The system detects statistical anomalies in burn-in telemetry that are associated with abnormal component behaviour. Whether a specific anomaly corresponds to a latent defect requires engineering judgment and, ultimately, validation on real hardware with known failure outcomes.

**Q: How do you prevent false negatives (missed defective components)?**
A: The system is explicitly tuned to minimize false negatives at the cost of higher false positive rate. The contamination parameter in Isolation Forest, the risk score thresholds, and the decision precedence rules are all calibrated conservatively. False negative rate is reported explicitly in model evaluation. Additionally, the hard safety rule engine provides a deterministic safety net for limit violations. We acknowledge that no system can guarantee zero false negatives.

**Q: Why Isolation Forest and not an Autoencoder?**
A: Isolation Forest is better suited for this problem because: (1) lot sizes are small (20–200 components), insufficient for training a meaningful Autoencoder; (2) Isolation Forest is interpretable — anomaly scores can be explained via SHAP; (3) it is computationally efficient for real-time inference; (4) it works well on tabular data without requiring deep learning infrastructure. An Autoencoder would add complexity without proportional benefit given the dataset size and interpretability requirement.

**Q: Why XGBoost and not LSTM for drift prediction?**
A: Each component has only 4 measurement checkpoints (0h, 24h, 96h, 168h). LSTM requires many time steps per sequence to learn meaningful temporal patterns — 4 steps is insufficient. XGBoost handles non-linear drift patterns effectively, provides feature importance for explainability, and is computationally efficient. If future deployments include higher-frequency measurements (e.g., hourly), temporal models would be reconsidered.

**Q: How do you handle synthetic data? Isn't it unreliable?**
A: Synthetic data is used for prototype development and methodology validation only. It is clearly labeled as SYNTHETIC throughout the system. We use physics-informed simulation (Arrhenius-inspired degradation models) to generate realistic drift patterns. We explicitly state that synthetic validation results do NOT constitute real-world ISRO hardware validation. The system requires validation on real burn-in telemetry before operational deployment. This is a standard approach in ML system development when real data is unavailable.

**Q: How do you validate the model on real hardware?**
A: The validation strategy for real hardware would be: (1) collect burn-in telemetry from a defined lot with known outcomes (components that subsequently failed in service vs. those that did not); (2) use this labeled dataset to evaluate Recall, Precision, FNR, and PR-AUC; (3) calibrate risk thresholds and fusion weights on this validated dataset; (4) conduct a prospective validation study before operational deployment. This is future work beyond the SIH prototype scope.

**Q: How do you explain an anomaly to an engineer?**
A: Every non-NORMAL decision generates a structured explanation containing: the observed value, the lot median, the deviation in sigma units, the temporal trend, the predicted 168h value with confidence interval, the top contributing features ranked by SHAP value, and a natural language narrative. The explanation cites specific evidence and never claims causality. Example: "Leakage current remained below the absolute limit but deviated substantially from the lot median, corresponding to a robust Z-score of 3.52, with a continuous increasing trend from 11.2 µA at 0h to 45.2 µA at 24h."

**Q: What happens if the AI system fails?**
A: The system is designed with a mandatory fail-safe principle: AI failure must never result in a silent dangerous PASS. If any AI service is unavailable, the decision engine returns MANUAL_REVIEW_REQUIRED. Hard safety rules execute independently of AI services and always take precedence. All failures are logged to the audit trail. The system fails loudly, not silently.

**Q: Can AI override mandatory safety limits?**
A: No. This is a hard architectural constraint. The hard safety rule engine executes first and independently. If a mandatory limit is violated, the decision is FAIL regardless of the AI anomaly score. The AI layer can only add flags (REVIEW, HIGH-RISK REVIEW) to components that pass hard limits. It cannot remove a FAIL decision produced by the safety rule engine.

**Q: How will this scale to real telemetry volumes?**
A: The architecture is designed for horizontal scalability. The ingestion, preprocessing, and inference services are stateless and can be scaled independently. For high-volume production, MQTT is replaced by Kafka (already designed for in the architecture). TimescaleDB handles time-series at scale with automatic partitioning and compression. The SIH prototype uses Docker Compose; production deployment uses a container orchestration platform.

**Q: What happens when component types differ across lots?**
A: The anomaly detector is trained per component type (and per lot where lot size permits). Lot-level statistics are computed separately per component type. The feature engineering includes component_type as an encoded feature. Cross-type comparison is not performed — a CMOS logic IC is not compared against a bipolar transistor.

**Q: How do you handle lot-to-lot variation?**
A: Lot-to-lot variation is handled by computing all population statistics (mean, median, MAD, percentiles) within each lot independently. The robust Z-score uses the lot's own median and MAD, not a global baseline. This means a component is flagged as anomalous relative to its own lot, not relative to a historical global average. This is the correct approach for lot-relative screening.

**Q: How do you prevent data leakage in model training?**
A: For drift prediction, the model is trained to predict value_168h from value_0h and value_24h only. The 96h and 168h values are never used as input features. Lot-level statistics used as features are computed on the training set only and applied to the test set — they are not recomputed on the test set. Train/test splits are performed at the lot level (not component level) to prevent information leakage between components in the same lot.

**Q: How do you monitor model drift in production?**
A: Input feature distributions are monitored using Population Stability Index (PSI) and KL divergence, compared against the training distribution. When PSI > 0.2 (configurable threshold), an alert is generated and predictions are flagged as POTENTIALLY_DEGRADED. Model output distributions (anomaly score distribution, prediction error on labeled samples) are also monitored. Retraining is triggered manually after engineer approval — not automatically.

---

## 44. LIMITATIONS & FUTURE SCOPE

### Current Limitations

1. **[SYNTHETIC DATA ONLY]**: All prototype validation uses synthetic data. Real-world performance is unknown until validated on real ISRO burn-in telemetry.
2. **Small lot sizes**: Statistical methods require minimum lot size (~10 components) for meaningful population comparison. Very small lots reduce detection power.
3. **4-checkpoint constraint**: Drift prediction uses only 4 checkpoints. Higher-frequency measurements would enable better temporal models.
4. **No physics model integration**: The system uses statistical/ML methods. Integration with physics-of-failure models (Arrhenius, Black's equation) would improve prediction accuracy.
5. **Threshold calibration**: All risk thresholds and fusion weights are prototype defaults. They require calibration on validated real data before operational use.
6. **No functional test integration**: The system analyzes parametric measurements only. Functional test results are not currently integrated.
7. **Single-parameter drift prediction**: Module B predicts one parameter at a time. Joint multi-parameter drift prediction is future work.

### Future Scope

1. Integration with real ISRO burn-in telemetry and validation study
2. Physics-of-failure model integration for improved drift prediction
3. Higher-frequency telemetry support (hourly measurements) enabling temporal neural models
4. Multi-parameter joint drift prediction
5. Automated threshold calibration using validated historical data
6. Integration with functional test results
7. Radiation effects correlation (for space-grade components)
8. Federated learning across multiple burn-in facilities
9. Digital twin integration for component-level simulation
10. Formal safety case documentation for operational deployment

---

## 45. CONCLUSION

This document describes a complete, production-oriented system architecture for AI-assisted anomaly detection in component burn-in and environmental stress screening. The system is designed around three core principles:

1. **Safety First**: Hard safety rules always take precedence. AI cannot override mandatory engineering limits. AI failure never results in a silent PASS.

2. **Explainability**: Every non-NORMAL decision is accompanied by an engineer-readable explanation citing specific evidence. The system augments engineer judgment; it does not replace it.

3. **Auditability**: Every decision has a complete, immutable audit trail. The system supports full decision reproducibility.

The key innovation is the clear distinction between three fundamentally different screening concepts:

```
STATIC SAFETY LIMIT CHECK
    "Does the value exceed the absolute datasheet limit?"
    → Deterministic, mandatory, always executes

        ≠

DYNAMIC BEHAVIOURAL ANOMALY DETECTION
    "Is this component behaving abnormally relative to its lot population?"
    → Statistical/ML, population-relative, assistive

        ≠

FUTURE DRIFT RISK PREDICTION
    "Based on early measurements, is this component likely to approach
     its limit before the end of burn-in?"
    → Predictive, uncertainty-quantified, assistive
```

The combination of all three in a single auditable engineering workflow — with deterministic safety precedence, AI-assisted behavioural screening, and predictive drift analysis — represents a meaningful advancement over traditional fixed-threshold screening.

**[FINAL DISCLAIMER]**: This is a prototype system developed for Smart India Hackathon. It requires validation on real ISRO burn-in telemetry, calibration of all thresholds and weights by qualified engineers, and formal safety case review before any operational deployment. All AI outputs are advisory. The qualified engineer makes all final component disposition decisions.

---

*Document Version: 1.0.0*
*Project: SIH26170*
*Organization: ISRO (Problem Statement)*
*Prepared for: Smart India Hackathon Technical Presentation*
*Data Classification: Prototype / Research*
*All synthetic data clearly labeled as SYNTHETIC throughout*
