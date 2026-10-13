# Capstone Project Plan: Distribution Grid Anomaly Detection Using Simulated SCADA Measurements

## Project Idea

In this project, you will build a data-driven anomaly detection system for a distribution power network. The goal is to simulate normal and abnormal operating conditions, generate SCADA-like measurements, train a machine learning model, and produce an alert when the system appears to be unhealthy.

The project focuses on **distribution networks**, not transmission networks. The emphasis is on Data Science: simulation-based dataset generation, time-series analysis, anomaly detection, model evaluation, and operator decision support.

## Main Objective

Develop a system that can answer the following question:

> Given real-time SCADA-like measurements from a distribution grid, can we detect whether the network is operating normally or under an abnormal/fault condition?

## Proposed Workflow

### 1. Use a Ready Distribution Network in pandapower

Start with a ready-made distribution network model in `pandapower`.

Possible examples include:

- CIGRE medium-voltage distribution network
- IEEE-style distribution feeder
- Any suitable built-in or publicly available distribution test network

The network should include typical distribution-grid components such as:

- buses
- lines
- transformers
- loads
- switches
- external grid/substation connection

### 2. Simulate One Year of Healthy Operation

Simulate the network for 365 days using a 5-minute sampling interval.

The number of time steps is:

```text
365 days x 24 hours/day x 12 samples/hour = 105,120 data points
```

During the healthy simulation, vary the operating conditions while keeping the system in a normal state.

Examples of healthy variations:

- daily load patterns
- weekday/weekend differences
- seasonal demand changes
- random demand variation
- optional PV generation variation
- optional temperature-related load variation

At each time step, run a power-flow simulation and store the relevant measurements.

### 3. Introduce Faults and Abnormal Events

After generating healthy operation data, deliberately introduce different types of abnormal events during the year and label them as anomalies.

Examples of grid-related anomalies:

- line outage
- transformer outage
- overloaded feeder
- voltage violation
- sudden abnormal load increase
- sudden PV generation drop
- switch status change
- partial feeder disconnection

Examples of measurement-related anomalies:

- missing SCADA measurement
- stuck sensor value
- noisy measurement
- false high/low voltage value
- communication dropout

For a cleaner first version of the project, focus first on **physical/grid anomalies**. Measurement anomalies can be added as an extension.

### 4. Generate SCADA-Like Measurements

At each time step, record measurements similar to what may be obtained from a SCADA system.

Possible measurements include:

- bus voltage magnitude
- feeder active power
- feeder reactive power
- line current
- line loading percentage
- transformer loading percentage
- switch or breaker status
- total system demand
- substation active/reactive power

The dataset should include both the simulated measurements and the system status.

Example dataset structure:

| Time | V_bus_1 | V_bus_2 | P_feeder | Q_feeder | Transformer loading | Status |
|---|---:|---:|---:|---:|---:|---|
| t1 | 0.99 | 0.98 | 2.4 | 0.7 | 65% | Healthy |
| t2 | 0.97 | 0.96 | 2.9 | 0.9 | 78% | Healthy |
| t3 | 0.88 | 0.85 | 3.5 | 1.2 | 115% | Anomaly |

### 5. Build the Training Dataset

The final training dataset consists of:

- input features: simulated SCADA-like measurements
- target label: healthy or anomaly

You may start with binary classification:

```text
0 = Healthy
1 = Anomaly
```

A more advanced version may classify anomaly type:

```text
0 = Healthy
1 = Line outage
2 = Transformer overload
3 = Voltage violation
4 = Sensor anomaly
```

### 6. Train an Anomaly Detector

Train one or more models to detect whether the grid is operating normally or abnormally.

Possible baseline models:

- Logistic Regression
- Random Forest
- XGBoost
- Isolation Forest
- One-Class SVM

Possible advanced models:

- LSTM autoencoder
- GRU model
- temporal CNN
- Transformer-based time-series model

Start simple, then improve.

### 7. Create a Realistic Test Dataset

Create a separate test dataset that mimics real-time SCADA measurements.

The test data should include:

- healthy periods
- fault/anomaly periods
- operating conditions not exactly identical to the training data

The test set should be used to evaluate whether the trained anomaly detector can generalize to new operating conditions.

### 8. Generate Operator Alerts

After training, the model should process incoming SCADA-like measurements and produce an alert when an anomaly is detected.

Example alert:

```text
Anomaly detected at 14:35.
Possible abnormal voltage/drop or feeder overload condition.
Recommended action: inspect feeder measurements and run detailed power-flow analysis.
```

## Data Splitting Strategy

Since anomalies are usually rare, do not split the data without checking the labels.

At minimum, use a stratified split so that both training and testing contain healthy and anomalous samples.

However, because this is time-series data, a simple random split can cause data leakage. For example, if one fault lasts for one hour, it creates 12 consecutive 5-minute records. A random split may place some records from the same fault event in training and others in testing, making the model look better than it really is.

Recommended approach:

### Event-Based Split

Split by fault event, not by individual rows.

If a fault event lasts from 10:00 to 11:00, all records belonging to that event should stay together in either the training set or the test set.

Add an `event_id` column:

| Time | Status | Event ID |
|---|---|---|
| 2026-01-01 10:00 | Healthy | none |
| 2026-01-05 14:00 | Anomaly | F001 |
| 2026-01-05 14:05 | Anomaly | F001 |
| 2026-01-05 14:10 | Anomaly | F001 |

This produces a more realistic evaluation.

Suggested evaluation design:

1. Use a stratified random split as a simple baseline.
2. Use an event-based split as the main realistic evaluation.
3. Optionally test on unseen fault types to evaluate stronger generalization.

## Evaluation Metrics

Use metrics that are suitable for imbalanced classification.

Recommended metrics:

- precision
- recall
- F1-score
- confusion matrix
- false alarm rate
- missed detection rate
- detection delay
- ROC-AUC or PR-AUC

For this project, recall is especially important because missing a true fault can be serious. However, false alarms should also be controlled because too many unnecessary alerts reduce operator trust.

## Expected Deliverables

By the end of the project, you should deliver:

1. A pandapower-based distribution network simulation.
2. A one-year time-series dataset of SCADA-like measurements.
3. A labeled anomaly dataset.
4. At least one trained anomaly detection model.
5. A realistic test evaluation.
6. A comparison of baseline and improved models.
7. A simple alerting interface or dashboard.
8. A final report explaining the methodology, results, limitations, and future work.

## Suggested Project Title

**Data-Driven Anomaly Detection in Distribution Networks Using Simulated SCADA Measurements**

Alternative titles:

- **SCADA-Based Fault Detection in Distribution Networks Using Machine Learning**
- **Simulation-Based Anomaly Detection for Distribution Grid Observability**
- **Machine Learning for Distribution Grid Health Monitoring Using Time-Series Measurements**

## Important Notes

This is not mainly a power-system protection design project. The main contribution should be Data Science.

The Data Science contribution includes:

- simulation-based dataset generation
- time-series feature extraction
- anomaly labeling
- imbalanced classification
- event-based train/test splitting
- model evaluation
- real-time alerting logic

The power-system simulation provides the environment, but the main project value comes from building and evaluating the data-driven anomaly detection pipeline.
