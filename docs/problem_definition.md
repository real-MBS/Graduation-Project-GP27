# GP27 Framework: Problem Definition & Objectives (v0.0.1)

## 1. Problem Statement
Electrical distribution networks frequently suffer faults on unmonitored downstream branches. While utilities receive Medium-Voltage (MV) SCADA measurements and incoming customer outage calls, they lack exact visibility into dynamic Low-Voltage (LV) switching states and precise downstream topologies. The problem is that existing rule-based systems struggle to fuse these two imperfect data sources (electrical physics and stochastic human reports) to accurately localize faults.

## 2. Reusable Framework Objective
The goal is to develop a reusable, configurable digital-twin-inspired framework that creates a utility-specific network replica, simulates component failures and stochastic customer reporting behavior, generates labeled feature datasets, and trains a supervised machine learning model to rank likely failure regions. 

## 3. Operational Phases
The framework operates in two distinct phases:
*   **Offline Simulation and Training:** Ingests a utility-specific configuration (network topology, sensor registry), simulates steady-state operating conditions and failures, generates a wide feature dataset, and trains a supervised ML model without leaking hidden ground truth[cite: 4].
*   **Operational Inference (Replayed):** Acts as a real-time adapter that takes observed MV SCADA data and timestamped customer calls at a specific decision cutoff time, applies the trained model, and returns ranked failure predictions[cite: 4].

## 4. Stakeholders
*   **Grid Operators:** Require rapid, ranked localization outputs to dispatch repair crews[cite: 4].
*   **Utility Engineers:** Configure the digital replica with specific network parameters, sensor locations, and legitimate LV switching configurations[cite: 4].
*   **ML Researchers:** Utilize the offline pipeline to benchmark different data-fusion algorithms[cite: 4].

## 5. Ranked-Region Output
Because exact LV cable locations cannot be guaranteed and multiple downstream faults exhibit identical upstream MV SCADA signatures, the model will output a Top-k ranking of distinguishable **service regions** rather than exact component coordinates[cite: 4].

## 6. Limitations and Assumptions
*   **LV Assumptions:** The system simulates valid, changing LV switch configurations, but these states remain strictly hidden from the ML model[cite: 4].
*   **Excluded Scope:** The framework does **not** perform LV topology reconstruction, state-estimation/observability studies, or short-circuit/protection transient modeling[cite: 4]. It does not utilize smart-meter data[cite: 4].
*   **ML Paradigm:** The target is strictly supervised fault localization, not healthy-only anomaly detection[cite: 4].