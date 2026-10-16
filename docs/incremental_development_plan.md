# GP27 — Digital-Twin-Based AI Framework: Incremental Development Plan

> **Status:** Tentative student roadmap. Increment numbering, exact network size, and scenarios may change following feasibility and supervisor review.
>
> **Project:** *Digital-Twin-Based AI Framework for Fault Localization in Electrical Distribution Networks*  
> **Companion specification:** `GP27_problem_definition_and_assumptions.md` (read before working on this plan).

## Scope and academic boundary

**Goal:** Develop a **reusable, configurable digital-twin-inspired framework** that creates a utility-specific network replica, simulates operating conditions/failures and report behavior, generates labeled datasets, trains a utility-specific supervised model, and then predicts and ranks likely failed components or distinguishable service regions **after customer outages**, from simulated MV SCADA-type measurements down to selected 11/0.4-kV transformer-area measurement points and synthetic customer reports. Simulate valid, changing LV switch configurations that are hidden from the ML model; retain known MV topology. Build on a ready **pandapower** network with a synthetic LV extension. **No smart-meter data, LV topology reconstruction, state-estimation/observability study, exact LV cable-location guarantee, or detailed short-circuit/protection transient modeling.** The target is supervised localization, **not healthy-only anomaly detection**. **The simulator and model-training pipeline are permanent framework components, not disposable data-preparation scripts.** The student version demonstrates replay-based inference; live synchronization/integration is future work.

**Milestones:** **GP1 ends at `v0.6.0-data`**, with the digital-replica and scenario/data-generation pipeline implemented, datasets prepared, features specified/produced, leakage-safe splits defined, and the ML approaches designed **but not implemented or trained**. **GP2 begins at `v0.7.0-model`** with implementation and training of the baseline and supervised ML models, followed by comparison, integration, full testing, and final release. A small **electrical/simulation proof of concept** is required during GP1 before committing to the final sensor set and localization granularity; this is **not an ML proof of concept**.

**Report alignment:** Ch. 1 (Introduction, problem, related work, contribution), Ch. 2 (plan, deliverables, tasks, timeline, roles, risks, costs, management), Ch. 3 (requirements), Ch. 4 (architecture, justified UML/system and physical design), Ch. 5 (data collection, profiling, feature engineering, loading), Ch. 6 (implementation), Ch. 7 (testing and evaluation), Ch. 8 (conclusion/future work), Appendices A–C. **Feasibility is a milestone in this plan, not a separately titled chapter of the report template.**

## Increment table

| Version | Type | Stage / increment | Main question | Expected output / review evidence | Report template | Suggested tools |
|---|---|---|---|---|---|---|
| `v0.0.1-design` | Analysis | Problem definition | What reusable utility-specific digital-replica-to-localization workflow is required, and at what fault-region resolution? | Problem statement, reusable framework objective, offline simulation/training and operational inference phases, stakeholders, ranked-region output, limitations and changing-LV assumptions. | Ch. 1.1–1.2 | Markdown, stakeholder notes |
| `v0.0.2-design` | Analysis | Related work and contribution | What is known about distribution digital twins/digital replicas, simulation-based fault localization, SCADA/call fusion and uncertainty? | Critical related-work table distinguishing operational digital twins from offline replicas, rule-based/supervised baselines, claimed reuse and data-fusion contributions. | Ch. 1.3–1.4 | Literature, Zotero |
| `v0.0.3-design` | Analysis | Feasibility study and proof of concept | Can valid LV switching and post-outage MV measurements distinguish a useful set of failure regions? | **Go/no-go demo:** small **configurable digital-replica proof of concept** on a ready pandapower network + LV extension, ≥2 valid LV configurations, ≥2 component failures, interrupted-customer identification, limited MV pre/post SCADA export; technical/data/operational/schedule/cost feasibility and risks; decide regional label granularity. | Ch. 2.2, 2.4–2.5; Ch. 3.2; input to Ch. 4–5 | pandapower, Python, pytest |
| `v0.0.4-design` | Analysis | Requirements and project plan | What inputs, outputs, functional/quality requirements and milestones must be met? | Stakeholders, numbered functional/nonfunctional requirements, deliverables, work breakdown, dependencies, Gantt/PERT, roles, risk register, costs, tooling. | Ch. 2; Ch. 3 | Markdown, Gantt, tables |
| `v0.0.5-design` | Design | Use-case modeling | How do operators/students simulate scenarios, train a model and inspect ranked candidates? | UML **Use Case Diagram** and concise descriptions of key use cases. | Ch. 4.2 | PlantUML, Draw.io |
| `v0.0.6-design` | Design | System architecture | How do **utility configuration → digital replica → offline simulation/dataset/training → versioned model → operational/replayed inference** interact? | **Block diagram based on the companion specification §2**, architecture diagram and UML **Component Diagram**; separate offline/periodic training from online/replayed inference, with explicit data contracts, utility-specific configurations and no hidden-ground-truth leakage. | Ch. 4.1–4.2 | Draw.io, PlantUML |
| `v0.0.7-design` | Design | Workflows | How do the offline simulation/training lifecycle and operational/replayed outage prediction proceed? | UML **Activity Diagram(s)** covering network setup → scenarios → data → training, and observed SCADA/calls → cutoff-aware inference → ranked regions; indicate future feedback/retraining. | Ch. 4.2 | PlantUML, Mermaid |
| `v0.0.8-design` | Design | Component interactions | Which modules exchange data during one simulated failure/training case and an operator localization request? | UML **Sequence Diagram** for inference with Top-k ranking, plus a concise offline scenario/training sequence if justified. | Ch. 4.2 | PlantUML, Mermaid |
| `v0.0.9-design` | Design | Software structure | Which project-owned packages/classes and interfaces are necessary? | UML **Package Diagram** and focused **Class Diagram** (network importer, replica/scenario engine, measurement registry, call generator, dataset builder, trainer/model registry, inference adapter, ranker); no unnecessary object/state diagrams. | Ch. 4.2 | PlantUML |
| `v0.0.10-design` | Design | Deployment design | Which nodes host replica/simulation, scenario data, model registry and replayed inference, and how would future utility interfaces attach? | UML **Deployment Diagram** separating training from inference; laptop/VM/server prototype, storage and versioned model artifacts. Show future utility SCADA/OMS as **external conceptual interfaces**, not implemented live integration. | Ch. 3.2; Ch. 4.2–4.3 | PlantUML |
| `v0.0.11-design` | Design | Network, LV switching and candidate regions | What network, legitimate switching states, faultable components and coarse labels exist? | Selected ready MV network plus configurable import contract, synthetic LV extension, network view, valid switch configurations, failure taxonomy, region map, consistency/versioning checks. | Ch. 4.3; Ch. 5.1 | pandapower, diagrams, YAML |
| `v0.0.12-design` | Design | MV SCADA measurement design | Which **specific** operator-visible MV quantities and locations are assumed? | Sensor registry `SCADA_M1...`, voltage/current/P/Q/status as justified, units, sample cadence, pre/post windows, missingness; distinguish physical simulator truth from usable model inputs. **Not an observability exercise.** | Ch. 3; Ch. 4.3; Ch. 5.1 | pandapower, Pandas, YAML |
| `v0.0.13-design` | Design | Failure and call scenario design | How will disconnections and probabilistic customer reports be generated? | Post-failure/isolation rules, interrupted-customer logic, Bernoulli/report-delay assumptions, possible false calls, cutoff times, random seeds; scenario matrix across LV configurations. | Ch. 4.3; Ch. 5.1 | Python, NumPy, YAML |
| `v0.0.14-design` | Design | Dataset/schema design | What records make up training samples and protected ground truth? | SCADA long table, calls table, restricted scenario labels, wide feature table and sample row; primary `y_region`, optional `y_failed_component`; data dictionary, no-leakage feature allowlist. | Ch. 4.3; Ch. 5.1–5.4 | Pandas, Parquet, Markdown |
| `v0.0.15-design` | Design | ML and test design | How will data fusion and topology generalization be established fairly? | B0 rule-based baseline; `MODEL_CALLS`, `MODEL_SCADA`, `MODEL_FUSION`; event/run/config-disjoint splits, Top-1/Top-3/macro-F1/calibration, report-delay and SCADA-missingness protocol. | Ch. 4.3; Ch. 6.3; Ch. 7.1 | scikit-learn, experiment tables |
| `v0.1.0-impl` | Implementation | Build and validate simulator | Does the chosen network function across legitimate LV configurations? | Reusable network-loader/replica adapter, configuration switcher, network/energized-connectivity tests, healthy flow, schema validation and deterministic seeds. | Ch. 6.1–6.2 | pandapower, pytest |
| `v0.2.0-data` | Data | Generate baseline MV electrical states | Can varied healthy loads produce reproducible SCADA measurement windows? | Pre-event healthy operating cases with selected **MV** sensor values, timestamps/units/quality, validation and generator manifests; no mandatory year-long or 5-min dataset. | Ch. 5.1; Ch. 6.2 | pandapower, Pandas, Parquet |
| `v0.3.0-data` | Data | Generate failures and post-event SCADA | Do component removals create well-defined service interruptions and valid post-event signatures? | Single-failure scenario generator, simplified isolation, interrupted-customer truth, selected MV post-event readings, explicit treatment of de-energized/invalid cases; true component and region metadata kept separate. | Ch. 5.1–5.2; Ch. 6.2 | pandapower, pytest |
| `v0.4.0-data` | Data | Simulate customer calls and measurement imperfections | Can realistic incomplete reports be generated for each simulated failure? | Customer-report table with known IDs/areas and call times; tunable reporting/delay/optional false calls; SCADA noise/missingness/latency where appropriate; reproducibility tests. | Ch. 5.1–5.2; Ch. 6.2 | NumPy, Pandas |
| `v0.5.0-data` | Data | Profiling and preprocessing | Are simulated distributions, labels and fields usable and well documented? | Data dictionary, quality checks, label balance, before/after preprocessing examples, training-only fitted transformers, invalid scenario audit. | Ch. 5.2; Ch. 6.2 | Pandas, matplotlib, sklearn |
| `v0.6.0-data` | Data / GP1 milestone | Feature engineering and leakage-safe splits | Can report and SCADA evidence be joined at a real decision cutoff without leaking hidden information? | Pre/post/delta SCADA features, area/time call features, feature allowlist, split manifest by run/event and held-out LV configurations; all event cutoffs share a split. **GP1 deliverable: reproducible simulated dataset and fully specified ML design; no trained ML model.** | Ch. 5.3–5.4; Ch. 7.1 | Pandas, sklearn, pytest |
| `v0.7.0-model` | Model / GP2 start | Baselines and single-source ML | How much can basic rules, calls alone and SCADA alone localize? | B0 explainable baseline; implemented `MODEL_CALLS` and `MODEL_SCADA` (e.g. logistic regression/random forest) and preliminary validation metrics. | Ch. 6.3; Ch. 7.1–7.2 | sklearn |
| `v0.8.0-model` | Model | Data-fusion fault localization | Does combining MV SCADA and calls improve region-ranking performance? | Implement `MODEL_FUSION`, Top-k output, saved pipeline/model, tuning on validation only; optional calibrated probabilities and a stronger model where justified. | Ch. 6.3 | sklearn, XGBoost (optional) |
| `v0.9.0-model` | Model | Initial GP2 model comparisons | Does fusion outperform B0/single-source methods under initial valid held-out cases? | Reproducible preliminary comparison table, confusion matrix, ablation, example ranked predictions, limitations, **GP2 preliminary model-comparison checkpoint**. | Ch. 6.3; Ch. 7.1–7.2 | Experiment logs, sklearn |
| `v0.10.0-model` | Model | Uncertainty and ambiguity analysis | When should the system present several plausible regions rather than one certain label? | Probability-calibration / Top-k and abstention study, analysis of indistinguishable failure candidates; revise label resolution if warranted. | Ch. 6.3; Ch. 7.2–7.3 | sklearn, plots |
| `v0.11.0-test` | Testing | Dynamic LV configuration generalization | What happens on LV switching configurations completely unseen in training? | Configuration-disjoint test performance, versus known configurations, per-region error analysis; scenario audit. | Ch. 7.1–7.3 | Python, sklearn |
| `v0.12.0-impl` | Implementation | End-to-end pipeline and operator output | Can a newly generated event flow to ranked region candidates? | Separate **offline simulate/train** and **replay/infer** CLI flows (or lightweight demo), persisted versioned model/configuration and useful evidence/uncertainty output; integration tests. | Ch. 6.2–6.4; Appendix A | Python, YAML, Streamlit (optional) |
| `v0.13.0-test` | Testing | Full frozen-set evaluation | How much does fusion help under changed report rates, cutoff times, missing SCADA and different loads? | Frozen test report: Top-1/Top-3, macro-F1, calibration, confusion matrices, error cases and baseline comparisons; no test-set tuning. | Ch. 7.1–7.3 | sklearn, Pandas, plots |
| `v0.14.0-test` | Testing | Robustness, scalability and system tests | Is performance stable under increasing cases and corrupted evidence? | Latency/resource measurements, SCADA missingness/noise, report sparsity/delay, repeated seeds, unit/integration tests, operational limitations. | Ch. 7.1–7.3 | pytest, profiling |
| `v0.15.0-docs` | Documentation | Final documentation and reproducibility | Can another team reproduce the project without live utility data? | README, setup, YAML examples, full data-generation recipe, editable UML + exports, report chapters, test results, user manual, change log and code documentation. | Ch. 1–8; Appendices A–C | Markdown, PlantUML, Sphinx/MkDocs |
| `v1.0.0` | Release | Final GP2 release | Can another team configure the framework for a network, generate fault scenarios, train its model and replay a localization request? | Tagged GitHub release: configurable digital replica, generators, network/sensor configs, reproducible sample data, versioned model and replay-based inference, tests, evidence, report and demo. | Ch. 1–8; Appendices | GitHub releases |

## GP1 / GP2 deliverable boundary

- **GP1 (through `v0.6.0-data`):** problem definition, related work, feasibility, requirements, project management, justified UML and architecture, digital-replica design and implementation, fault/customer-call/SCADA simulation and data preparation, documented feature engineering and splits, **proposed** ML methodology and evaluation protocol. Simulation code and data-generation evidence are required; **ML training, model implementation, and model-performance claims are not required**.
- **GP2 (starting `v0.7.0-model`):** implement the rule-based and ML baselines, train the single-source and fusion localization models, calibrate/rank candidate regions, evaluate against held-out configurations, integrate operational replay, and complete system testing and documentation.
- The official project report chapters are cumulative: GP1 may design items whose implementation/results are completed in GP2. The GP1 report must distinguish **planned model work** from implemented simulation/data-pipeline work.

## Reporting and diagram requirements

- **Ch. 1:** motivation, precise target/limitations, critically compared prior work and claimed contribution; distinguish simulated assumptions from verified utility facts.
- **Ch. 2:** deliverable/task table, schedule/Gantt and PERT (as required), ownership, task dependencies, risk assessment and estimated costs. Put the feasibility analysis here and cross-reference its technical proof of concept in Ch. 4–6.
- **Ch. 3:** explicit stakeholders, numbered essential/recommended functions, inputs/outputs, platform constraints, nonfunctional requirements (reproducibility, latency, robustness, data quality).
- **Ch. 4:** appropriate **Use Case, Component, Activity, Sequence, Package, focused Class, and Deployment diagrams**; **block diagram from companion specification §2**, separate offline-training and inference paths, utility-specific configuration boundary, high-level architecture plus network and sensor/feature design. Keep editable diagram source and exported image. Do **not** require Object or State Transition diagrams merely to fill a checklist.
- **Ch. 5:** simulated data-source description, schema, metadata, examples, profiling, preprocessing, feature derivation and storage; clearly protect labels and hidden LV state.
- **Ch. 6:** documented simulation/pipeline/model implementation, choices and rationale, critical algorithms, code structure, what was implemented/deferred.
- **Ch. 7:** predeclared metrics, leakage-safe splitting, model and system tests, actual results and failures; include unseen-LV-configuration test.
- **Ch. 8 and appendices:** conclusions/limitations/future work; installation and user guide, **changes from GP1**, and code documentation.

## Implementation, collaboration and version control

- Each increment produces reviewable outputs under `docs/`, `src/`, `tests/`, `configs/`, or `data/` as appropriate. Save dataset-generation configs and manifests; avoid committing very large generated datasets to Git.
- Use a separate feature branch for **every increment** (e.g. `feature/problem-definition`, `feature/feasibility`, `feature/customer-call-generator`, `feature/fusion-model`). Every student, **including the designated merger**, works on a feature branch and raises a pull request to `main`.
- A designated student is responsible for merges **after review**. Nobody develops or pushes directly to `main`. If increments proceed in parallel, isolate independent work and resolve shared contracts before merging.
- Apply the increment tag **to the approved merged commit on `main`**, e.g. `v0.0.3-design`, `v0.4.0-data`, `v0.9.0-model`. Do not tag an unmerged branch; maintain the version/evidence mapping in the change log.
- Version suffixes (`-design`, `-data`, `-model`, `-impl`, `-test`, `-docs`) are team naming conventions rather than strict semantic-version precedence. Optional increments may be altered by supervisor agreement without rewriting past tags.

### Suggested repository organization

```text
GP27/
├── README.md
├── docs/
│   ├── problem_definition_and_assumptions.md
│   ├── incremental_development_plan.md
│   ├── feasibility/
│   ├── requirements/
│   ├── architecture/
│   │   └── diagrams/           # Editable UML sources and exported figures
│   ├── experiments/
│   └── report/
├── configs/
│   ├── utility_profile.yaml      # Reusable framework, utility-specific inputs
│   ├── network.yaml
│   ├── lv_configurations.yaml
│   ├── scada_registry.yaml
│   ├── call_simulation.yaml
│   └── experiments.yaml
├── src/
│   └── gp27/
│       ├── network/             # Digital replica / utility network import
│       ├── simulation/          # Switching + failure scenario engines
│       ├── measurements/
│       ├── calls/
│       ├── data/
│       ├── features/
│       ├── models/              # Training + versioned model artifacts
│       └── inference/           # Replayed/live-input-compatible prediction adapter
├── tests/
├── data/
│   ├── samples/               # Tiny safe-to-commit examples
│   ├── generated/             # Normally gitignored
│   └── manifests/
├── models/                     # Large trained artifacts normally gitignored
└── pyproject.toml
```

## Framework acceptance criteria and feasibility safeguards

- **Reusable framework:** demonstrate that network, sensor locations, LV configurations, report parameters and candidate regions are configuration inputs, not hard-coded throughout the ML pipeline.
- **Two-phase operation:** deliver a reproducible **configure/simulate/train** command sequence and an independent **load-model/replay-observations/rank-regions** path.
- **Model provenance:** save which network, sensor inventory, feature schema, scenario generator and training split produced each model.
- **Digital-twin terminology:** student artifact is a *digital-twin-inspired prototype*; actual continuous field synchronization, validated utility calibration and field integration are future work.

## Evaluation and feasibility safeguards

1. **Feasibility before scale:** approve the fault taxonomy, coarse labels, LV switching states and sensor registry only after validating the small electrical prototype.
2. **No privileged features:** true failed component, actual disconnected-customer list, hidden LV switch configuration and post-cutoff data must not enter model features.
3. **No invented telemetry:** model receives only explicitly simulated **available** MV SCADA channels. De-energized/undefined states have an explicit quality/missing status.
4. **Customer calls are stochastic:** call model parameters are assumptions, not utility statistics. Test varied reporting levels, false calls and delays.
5. **Appropriate physics claims:** pandapower steady-state/isolation scenarios do not by themselves reproduce fault transients or detailed protection coordination.
6. **Split carefully:** all cutoffs and variants of one event stay together; reserve independent runs and unseen LV configurations; choose preprocessing/calibration/hyperparameters without peeking at frozen test data.
7. **Measure improvement, not just accuracy:** assess how much combining SCADA + calls improves over source-only models and a simple rule baseline; report cases that cannot be uniquely localized.

**Supervisor checkpoint:** Review the `v0.0.3-design` feasibility outcome before authorizing generation of a large dataset. If the available SCADA and call evidence cannot distinguish individual LV components, set the primary target to **service regions** and document the resolution limit.
