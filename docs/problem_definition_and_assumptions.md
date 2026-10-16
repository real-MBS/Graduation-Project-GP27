# GP27 — Digital-Twin-Based AI Framework for Fault Localization in Electrical Distribution Networks

> **Status:** Tentative project specification for Data Science graduation students; assumptions and scenario boundaries are to be validated in the feasibility increment.
>
> **Project title:** *Digital-Twin-Based AI Framework for Fault Localization in Electrical Distribution Networks*

## 1. Practical motivation and problem statement

When customers lose electricity, a utility needs to identify the likely failed distribution-network component or affected service region quickly enough to dispatch an appropriately specialized crew. Customer calls alone may suggest an outage cluster, but incomplete reporting and changing LV connections make it difficult to pinpoint the failure. Electrical measurements from the medium-voltage (MV) network, extending to selected measurement points at/near 11-kV/0.4-kV distribution transformers, can provide independent evidence.

**Objective:** Develop and evaluate a **configurable, digital-twin-inspired simulation and supervised-ML framework**, rather than a one-off synthetic dataset or a single network-specific model. A utility should be able to configure a replica of its distribution network, simulate fault and reporting scenarios, train its own localization model, and use that model to rank failure regions during outages. The localization subsystem is a supervised ML decision-support system that **ranks plausible failed components or service regions and communicates uncertainty**, by combining (i) simulated MV-level SCADA-type measurements, (ii) simulated customer outage reports, and (iii) the known MV network structure. The system should not claim to identify an exact LV cable when available observations do not make that possible.

**Primary research question:** Does combining MV electrical measurements with incomplete customer outage reports improve post-outage fault localization compared with either source alone, particularly when the LV network is reconfigured and its exact connectivity is unavailable to the model?

**Intended outputs:** (1) A reusable network-replica/scenario/data-generation/training pipeline that can be configured for a utility, and (2) an operator-facing inference component that, at a chosen decision time after an outage begins, returns Top-k component/region IDs, calibrated probabilities where supported (otherwise scores), supporting evidence, and an optional suggested crew category based on the predicted region (not crew scheduling).

## 2. Intended operational concept and candidate architecture

This section describes the **expected lifecycle if a distribution utility adopts the framework**. It is intended to serve as the narrative basis for a future **block diagram, UML Component Diagram, Activity Diagram, and deployment architecture**. The student demonstrator will use simulated sources and offline files; **real-time deployment, field synchronization and utility acceptance are future work**.

### 2.1 Two connected operational phases

**Phase A — Configure the digital replica, simulate and train (offline / periodically repeated)**

1. **Utility network configuration:** Supply available GIS and engineering records for MV feeders, switches, lines, 11/0.4-kV transformers, electrical parameters, and the SCADA sensor registry. Supply synthetic or authorized customer identifiers and reported areas. Do not assume a complete LV single-line diagram.
2. **Create a configurable digital replica:** Import the utility-specific network into the simulator (pandapower adapter). Add a *synthetic, plausible LV extension* and a documented set of physically valid LV switching configurations where actual LV connections are unknown. Distinguish known utility facts from synthetic assumptions.
3. **Generate operating scenarios:** Vary loading and plausible switching states, including daily LV reconfiguration, while checking network connectivity and solver validity.
4. **Simulate failure scenarios:** Select candidate failed components, apply simplified isolation/repair-status assumptions, derive interrupted customers, and compute valid pre-/post-outage **MV SCADA-type signals only at configured measurement locations**.
5. **Simulate incoming outage reports:** Generate call identities/areas and arrival times probabilistically from the interrupted-customer set; vary non-reporting, call delays and optional false calls.
6. **Construct labeled datasets:** Join timestamp-valid SCADA and customer reports at defined decision cutoffs. Keep actual failed components, interrupted-customer lists and hidden LV switch configurations in protected simulation metadata, never in prediction features.
7. **Train, validate and register a utility-specific model:** Compare an explainable baseline, calls-only, SCADA-only and fused models. Evaluate ranking, uncertainty and robustness on independent simulations and **unseen LV configurations**; save the selected model together with its feature schema, sensor registry, network version and simulation assumptions.
8. **Update and retrain when needed:** If the utility's documented MV network, sensor set or operating conditions change, update the replica, regenerate relevant scenarios and revalidate/retrain. This update is part of the intended framework lifecycle, not evidence of a live synchronized twin in the student prototype.

**Phase B — Locate a reported outage (operational inference)**

1. **Receive observations:** At prediction time, receive real utility SCADA readings from the configured MV sensors and outage calls with customer IDs/known geographical areas and timestamps. In the student demonstrator, replay held-out *simulated* observations through the same interface.
2. **Validate and align:** Check sensor identity, units, missingness, stale values, event/call deduplication and observation cutoff. Do not access the simulator's failure label, hidden LV connectivity or future calls.
3. **Apply the saved preprocessing and trained model:** Construct the expected features, score plausible failure regions/components, and return a Top-k ranked list with uncertainty and relevant supporting inputs.
4. **Support the operator:** Display likely failed region(s), evidence quality and an optional **crew specialization category**. The operator makes the dispatch decision; scheduling and automated network control are out of scope.
5. **Close the learning loop (future utility operation):** When field crews verify the actual failure, capture the confirmed label and assess model errors; use confirmed cases to improve calibration, validate simulation assumptions and plan retraining. Real confirmed outage records are **not required** for GP27 students.

### 2.2 Candidate block-diagram components and interfaces

The following is a **logical flow specification**, not a claim that production integrations will be implemented.

```text
CONFIGURATION AND TRAINING (offline / periodic)
Utility MV/GIS records + sensor registry + customer areas
                          |
                          v
              Network digital replica
                 (pandapower adapter)
                          |
           operating + switching scenarios
                          v
         Failure / isolation scenario engine
                 /                     \
                v                       v
       Simulated MV SCADA       Interrupted-customer truth
                                        |
                                        v
                               Customer-call generator
                 \                     /
                  v                   v
                    Labeled dataset builder
                           |
                           v
                Feature + split pipelines
                           |
                           v
                   Train / validate ML
                           |
                           v
              Versioned utility-specific model
                           |
                           v
OPERATIONAL INFERENCE (replayed data in GP27; live in future)
Available MV SCADA + received customer outage calls
                           |
                           v
           Quality checks + cutoff-aware features
                           |
                           v
                Saved localization model
                           |
                           v
             Ranked fault regions + uncertainty
                           |
                           v
                Utility operator / crew decision
```

| Logical component | Main inputs | Main outputs / boundary |
|---|---|---|
| Network importer / replica | MV model, components, electrical parameters, candidate LV configurations | Configured simulator; retains hidden physical-network truth |
| Scenario and failure engine | Replica, loading, valid switch states, failure scenarios | Interrupted loads/customers and pre/post simulated states |
| SCADA emulator / registry | Simulator states and **configured** MV sensor locations | Timestamped, qualified MV measurements; **not** every internal solver result |
| Call generator | Interrupted-customer truth, reporting/delay parameters | Timestamped reported customer IDs/areas, no non-caller ground-truth features |
| Dataset + feature pipeline | SCADA and calls by cutoff; separate scenario labels | Training matrix and protected target labels; no future/hidden-feature leakage |
| ML training / model registry | Training/validation data, approved feature schema | Utility-specific ranked-region model and version metadata |
| Inference adapter | Observed SCADA + real/replayed calls | Ranked regions, uncertainty and data-quality explanations |
| Operator interface | Ranked candidates, evidence, optional region-to-crew mapping | Decision support, **not** autonomous dispatch |

### 2.3 Digital-twin terminology and deployment limitations

This is a **digital-twin-based / digital-twin-inspired prototype** because the reusable replica is central to scenario simulation and utility-specific model training, and the architecture permits periodic alignment with utility records. It is **not yet a fully operational, continuously synchronized digital twin**: GP27 does not implement live two-way synchronization, a real SCADA/OMS connection, authenticated utility interfaces, or validation against actual utility outage histories. For real adoption, the network model and synthetic LV alternatives must be calibrated with available records and observed data; trained uncertainty reflects the simulated assumptions until field validation occurs.

### 2.4 Architecture and reusability requirements

- Keep **utility-specific configuration** (network files, valid switch states, SCADA point registry, customer-area catalog, candidate regions, scenario parameters) separate from **reusable framework code**.
- Provide configurable adapters and versioned input/output contracts for simulation, measurement extraction, call generation, dataset creation, training and prediction.
- Support swapping in another ready network and retraining a distinct model *without changing the core code*. Demonstrating this on a second small test network is a desirable GP2 extension; not a GP1 prerequisite.
- Record network/sensor/configuration versions and random seeds so that model outputs can be traced to their simulated training environment.

## 3. What the project does and does not do

### Included

- A **reusable, configurable digital-replica and model-training framework** that can be adapted to a utility-specific network (demonstrated using a ready-to-use **pandapower** distribution network, adapted with manageable MV feeder sections, 11/0.4-kV transformers and a **synthetic LV extension** containing synthetic customers and switchable LV connections).
- A small library of **physically valid** LV switching configurations, possibly changed daily; the simulator retains the true configuration, but the ML model does **not** receive undisclosed LV connectivity.
- Post-failure outage scenarios: a line section, transformer or LV supply branch becomes unavailable, with any protection/isolation behavior stated explicitly and simplified.
- Healthy pre-event and post-event **MV SCADA-type** measurements from an **assumed fixed and explicitly enumerated sensor set**, including permitted measurement points near transformer MV terminals. Include equipment/switch indications only if they would realistically be reported to an operator.
- Synthetic customer calls: customer ID or address/area, call timestamp, affected/not affected ground truth known only to simulator, probabilistic non-reporting and delays, and optionally false reports.
- Supervised classification/ranking of candidate **failed components or service regions**, baseline comparisons, uncertainty, and robustness/generalization evaluation.

### Excluded

- Smart-meter readings, smart-meter voltage/energy profiles, or meter-event telemetry.
- LV single-line diagram reconstruction, explicit topology estimation, state estimation and observability analysis.
- Short-circuit waveforms, transient fault signatures, detailed relay coordination, and protection-system design.
- Guaranteed exact identification of individual LV cables, and automated crew dispatch optimization.
- Live utility integration, continuous synchronization or field-validated digital twin operation; no claim that the virtual SCADA coverage matches actual deployed instrumentation.

## 4. Assumptions and safeguards

| ID | Assumption | Consequence / validation requirement |
|---|---|---|
| A1 | A ready distribution test network and its electrical parameters are supplied by pandapower (or adapted from an existing example). | Students are not asked to derive electrical models from scratch. |
| A2 | MV topology and 11/0.4-kV transformer identities are known to the utility at the modeled observation time, unless an experiment explicitly varies known MV topology. | MV topology can enter the model; not the hidden LV switch state. |
| A3 | The exact LV wiring/SLD is **not available to the ML system** and LV switching can alter customer-to-branch connectivity, potentially daily. | Simulator knows the true network; operator-facing input excludes that ground truth. |
| A4 | Synthetic customer IDs and their reportable address/area are available for received calls. | Geographical call clusters are usable; a current LV branch/customer mapping is **not** assumed known. |
| A5 | Sensor locations and measured quantities on the MV network, potentially at selected transformer MV terminals, are **defined in advance**. | The simulator may calculate all bus values, but the model receives **only assumed SCADA points**, not hidden internal results. |
| A6 | SCADA values are timestamped, sampled at a declared cadence and may contain noise, latency, missing data, or de-energized/unavailable readings. | Sensor realism is part of generation; no universal full-bus sensing assumption. |
| A7 | Start with **one failed component per event**; use a small, predefined list of failure classes and valid switching states. | Multiple simultaneous component failures are outside the mandatory scope. |
| A8 | Each failure causes a clearly defined post-isolation state, and the simulator knows exactly which customers are interrupted. | `pandapower.runpp()` describes supported steady-state network states, **not the physical short-circuit or protective trip transient**. |
| A9 | Customer calls are generated from outage ground truth using an explicit stochastic behavioral model with tunable reporting probability and delay. | Calls are a synthetic assumption, not validated utility call statistics unless external data become available. |
| A10 | Every training example has a known simulation ground-truth failed component. | Fine-grained targets can be regrouped into indistinguishable service regions for fair evaluation. |
| A11 | A distinct *healthy/pre-event* operating condition is generated under varied loads before each fault. | Pre/post MV measurements and changes can be used if their timestamps precede the prediction cutoff. |
| A12 | A case is discarded or flagged when the post-event power flow fails, islands are handled incorrectly, or measurements are physically invalid. | Never treat a solver's failed result as a legitimate voltage/current reading. |
| A13 | Data splits are grouped by independent simulation run and topology/configuration; unseen LV configurations are reserved for a generalization test. | Prevents optimistic leakage from near-duplicate scenarios. |
| A14 | Model confidence is a prediction under the simulation assumptions, not proof of an actual utility fault location. | Show Top-k candidates, ambiguities and limitations. |

## 5. How the simulations work

1. **Initialize:** Load a valid MV/LV distribution model, sensor inventory, synthetic customers and a configuration of permitted LV switches.
2. **Choose the daily LV configuration:** Select a valid switching state, recording its ID **in metadata only**, hidden from model features.
3. **Set a healthy operating condition:** Sample realistic time-varying demands and compute healthy power flow; retain only operator-available measurements.
4. **Choose a single failed component:** Simulate its isolation/removal and the documented switching/protection response. Determine energized connectivity and which synthetic customers lose supply. Recompute valid energized islands where applicable.
5. **Collect post-event SCADA:** Export only configured MV/transformer-area sensor channels, with realistic status, noise, missingness and timestamps. For de-energized islands, explicitly mark measurements unavailable/de-energized rather than inventing solved voltages.
6. **Generate calls:** For each interrupted customer, independently or under a richer specified behavioral process, decide whether they report and when; optionally introduce a small false-report background rate.
7. **Freeze an observation cutoff:** Assemble only readings and calls available by e.g. 5, 15 or 30 minutes after the modeled outage start; no information from the future.
8. **Label:** Store true failed component and its acceptable coarse service-region label separately from ML inputs.
9. **Repeat:** Vary operating load, component, configuration, call behavior, SCADA quality, and random seed, retaining independently reproducible scenario IDs.

**Important:** Switch topology alone can determine affected customers in the simplest post-isolation cases. Power flow is still needed to generate valid **MV electrical operating signatures** for energized portions and variations in healthy/post-event quantities. Whether those signatures meaningfully distinguish candidates is an explicit **go/no-go feasibility test**, not an assumed result.

## 6. Proposed data model

**Unit of observation:** One event at one prediction cutoff. An event may produce several snapshots (5, 15, 30 minutes); these snapshots must remain in the **same data split**.

Use three linked tables plus a model-ready feature matrix:

1. `scada_timeseries`: `(scenario_id, timestamp, measurement_id, value, unit, quality_flag)` with a separate `sensor_registry` mapping each `measurement_id` to a physically plausible MV-side point.
2. `customer_calls`: `(scenario_id, call_id, customer_id, customer_area, call_timestamp, report_type)`; contains **only actual simulated incoming calls**. Non-callers do not appear in this table.
3. `scenario_labels`: `(scenario_id, failure_time, failed_component_id, failed_component_type, failure_region_id, hidden_lv_config_id, generator_seed, split_id)`; **labels and simulator metadata, not model input**.
4. `model_features`: a single row per `(scenario_id, decision_cutoff_min)` with aggregated timestamp-valid SCADA and call features, and training label(s). Feature columns are generated by joining only legally available information.

### 6.1 Illustrative SCADA measurement registry

The identifiers below are **example feature names, not verified utility sensor inventory**. A channel is only included if that sensor is assumed installed. Transformer measurements refer to **selected MV terminals / feeder instrumentation**, not smart meters or mandatory sensors at every transformer.

| ID | Example measurement location | Channel | Unit | Role |
|---|---|---|---|---|
| `M1` | MV feeder F1 head | RMS voltage magnitude | kV | Upstream supply status |
| `M2` | MV feeder F1 head | RMS line current | A | Feeder demand/interruption signature |
| `M3` | MV feeder F1 head | Active power | MW | Feeder load change |
| `M4` | MV feeder F1 head | Reactive power | MVAr | Companion load-flow information |
| `M5` | Selected MV section S1 | RMS voltage magnitude | kV | Helps separate upstream vs downstream disruptions |
| `M6` | Selected MV section S1 | RMS line current | A | Section load/interruption evidence |
| `M7` | MV terminal of transformer T1 | RMS voltage magnitude | kV | Local transformer-area supply indication |
| `M8` | MV terminal of transformer T1 | RMS line current | A | Local transformer-area load signature |
| `M9` | MV terminal of transformer T2 | RMS voltage magnitude | kV | Comparison across transformer areas |
| `M10` | MV terminal of transformer T2 | RMS line current | A | Comparison across transformer areas |
| `S1` | MV feeder breaker F1 | breaker status | 0/1/unknown | **Optional**: only if remotely observable |
| `S2` | Selected MV switch | switch status | 0/1/unknown | **Optional**: only if remotely observable |

For each analog `M#`, potential engineered columns include `M1_pre`, `M1_post`, `M1_delta`, `M1_missing` and `M1_age_sec`. The actual sensor list must be approved in the feasibility study. Real-world availability of MV-to-transformer SCADA is **an explicit project assumption**, not an established fact from the engineer.

### 6.2 Customer call features

| Feature | Meaning |
|---|---|
| `calls_total` | Number of outage reports received before the cutoff |
| `calls_first_delay_min` | Elapsed time from assumed incident start to first call, if any |
| `calls_last_delay_min` | Elapsed time to most recent call |
| `calls_area_A`, `calls_area_B`, ... | Counts of reports by **known customer address/area**, not an assumed current LV branch |
| `calls_unique_customers` | Distinct callers; prevents duplicated reports from dominating |
| `calls_spread_areas` | Number of distinct areas with at least one report |
| `calls_rate_5min` | Call count during the most recent five minutes before cutoff |
| `call_customer_ids` | Optional sparse/multi-hot representation of caller identities if the synthetic customer universe is fixed |

Do **not** construct model features such as `true_customers_affected`, `actual_outage_extent`, `failed_component_id`, or `hidden_lv_config_id`: these use ground truth or unknown LV connectivity. Likewise, the exact failure timestamp is not necessarily available operationally; if features are defined relative to it, justify a modeled detection/incident-start timestamp that would be available to the operator.

### 6.3 Example *wide* model-ready dataset

The following figures are **invented to illustrate schema only**; they are not pandapower outputs. For compactness only `M1` and `M2` are expanded. A real file would include applicable pre/post/delta/quality columns for **all selected SCADA measurements M1–M10**, additional customer-area counts, missing-data flags, and optional permitted equipment statuses.

| event_id | cutoff_min | M1_pre_kV | M1_post_kV | M2_pre_A | M2_post_A | calls_total | calls_area_A | calls_area_B | calls_unique_customers | y_failed_component | y_region |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|---|
| E0001 | 15 | 11.02 | 10.98 | 210 | 145 | 8 | 7 | 1 | 8 | MV_L03 | F1_downstream_section |
| E0002 | 15 | 11.01 | 11.00 | 185 | 178 | 3 | 3 | 0 | 3 | LV_B12 | T1_service_area |
| E0003 | 15 | 11.03 | 10.99 | 225 | 172 | 12 | 2 | 10 | 11 | TR_T2 | T2_service_area |
| E0004 | 30 | 10.99 | 10.99 | 195 | 194 | 1 | 0 | 1 | 1 | LV_B21 | T2_service_area |

Here `y_failed_component` is the simulator's ground-truth fine label and `y_region` is a coarser supervised response that may be more reliably identifiable. In training, select the primary response and document whether fine labels are used only for diagnostic evaluation. IDs should remain stable across simulation runs, and train/test separation must be by **event** (and the topology holdout), not individual rows.

### 6.4 Suggested model responses

- **Primary target:** `y_region` — a stable candidate service region or distinguishable MV/transformer-area group; output Top-1 and Top-3 candidates with scores or calibrated probabilities.
- **Secondary/optional target:** `y_failed_component` — only for component classes where the observations plausibly distinguish individual faults, and with explicit caveats.
- **Operator display:** Ranked candidates, uncertainty/ambiguity flag, summary of reports and relevant SCADA deltas, and optional fault-region-to-crew-type mapping.

## 7. Customer-call simulation without a specialized tool

For a customer `i` interrupted by a simulated component failure, generate a Bernoulli report indicator with probability `p_report`; for reporting customers, draw a nonnegative call delay (e.g. lognormal or gamma). Example: 30 affected customers at `p_report=0.4` yield 12 **expected** reports, not exactly 12. Reporting probabilities can differ by customer/area/time; a small false-report process can be added to unaffected customers. Calls arriving after a cutoff must be excluded at that cutoff.

Use standard Python/NumPy random generators and preserve parameter settings and seeds. **Do not present these distributions as known facts about the utility**; sensitivity testing across plausible rates/delays is required.

## 8. Models, baselines and evaluation

| System | Allowed inputs | Purpose |
|---|---|---|
| `B0` | Reports + known MV topology / area map | Explainable call-clustering or region-prior baseline |
| `M1` | Customer reports only | Quantify information from reports |
| `M2` | MV SCADA measurements only | Quantify electrical measurement contribution |
| `M3` | Reports + MV SCADA | Main data-fusion model |
| `M4` (optional) | M3 + legitimate MV-topology features | Assess topology-aware feature engineering |

**Warning on names:** `M1` and `M2` in the **SCADA registry** name measurement channels; `M1` and `M2` in this **model comparison** are experiment labels. In code, prefer `SCADA_M1`, `SCADA_M2`, etc. and `MODEL_CALLS`, `MODEL_SCADA`, `MODEL_FUSION` to avoid collision.

**Evaluation:** Top-1 and Top-3 region accuracy (Top-3 only where at least three candidates exist), macro-F1 and confusion matrix, region-level vs fine-component results, calibration (e.g. Brier score or reliability plot), ambiguity coverage, performance versus report probability/delay, SCADA missingness, and LV switching changes. Compare against B0; split by independent simulation runs/events with an **entirely unseen LV configuration** holdout; do not leak event variants across partitions. Test at multiple observation cutoffs to study how evidence accumulates.

**Identifiability rule:** If two faults under realistic operating conditions produce indistinguishable permitted SCADA and call patterns, the model is not expected to distinguish them. Merge them into justified service regions for the primary task or explicitly report their ambiguity rather than claim exact LV cable localization.

## 9. Feasibility gates before expanding the data generator

1. **Network gate:** Can a modest ready pandapower model and synthetic LV extension express multiple physically valid LV switch states and safely identify disconnected customers?
2. **Measurement gate:** Can the chosen **limited** MV SCADA inventory produce valid, materially varying pre/post-failure electrical signatures without using hidden simulated quantities?
3. **Information gate:** Do at least some competing fault regions produce measurably different SCADA+call evidence? If not, coarsen the labels or redesign the assumed sensor layout (a modeling decision, **not** an observability study).
4. **Data gate:** Can the team reproducibly produce hundreds/thousands of independent labeled fault events on available hardware, with controllable calls, measurements and switching scenarios?
5. **Evaluation gate:** Are event/run/configuration-disjoint validation and test partitions feasible with enough labeled examples per region?

## 10. Student-facing deliverables

- Documented assumptions, selected network, MV sensor registry and candidate failure/region taxonomy.
- Reproducible simulation of healthy states, post-disconnection measurements, LV switching configurations and interrupted-customer sets.
- Reproducible customer call generator and clean linked datasets/data dictionary.
- Leakage-safe ML feature pipeline and simple B0 baseline plus supervised models.
- Evaluation of M1/M2/M3, changing topology and uncertainty; short operator-oriented candidate-ranking demo.
- Source code, tests, editable UML diagrams, report sections and documented limits.

**Academic boundary:** GP1 covers problem definition, feasibility, architecture and UML design, an implemented digital-replica/simulation and synthetic-dataset pipeline, feature engineering, and **ML model/evaluation design only**. **ML model implementation and training begin in GP2**, followed by comparison, inference integration, testing and final release. The companion incremental plan maps the work to the official graduation-project template.
