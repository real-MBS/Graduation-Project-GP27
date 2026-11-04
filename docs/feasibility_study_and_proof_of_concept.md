# v0.0.3 - Feasibility Study and Proof of Concept

**Project:** Digital-Twin-Based AI Framework for Fault Localization in Electrical Distribution Networks  
**Milestone:** `v0.0.3-design`  
**Status:** ✅ Approved (Go)  

## 1. Executive Summary & Go/No-Go Decision
The objective of this proof of concept (PoC) was to validate the core physical and data-generation assumptions of the GP27 digital-twin framework before committing to large-scale dataset generation. We built a small, configurable digital replica, introduced valid Low-Voltage (LV) switching configurations, simulated component failures, and generated restricted post-outage MV SCADA measurements alongside stochastic customer calls.

**Decision:** **GO**. The framework successfully produces distinct, materially varying SCADA and call evidence for competing fault regions without leaking hidden simulation ground truth.

## 2. Digital Replica Setup
*   **Grid Model:** IEEE 33-bus radial distribution network (`pandapower.networks.case33bw()`).
*   **Solver Configuration:** Applied Newton-Raphson power flow (`pp.runpp`) with `check_connectivity=True` to explicitly calculate unsupplied buses and drop de-energized islands natively, generating valid missing-data states instead of solver crashes.
*   **Visualization:** Generated network topologies mapping bus indices to verify upstream/downstream relationships physically.

## 3. Scenarios Tested
To prove the framework can handle dynamic grid states, we tested combinations of failures and hidden LV switching configurations:

*   **Failure 1 (Main Trunk - Node 10):** Isolated Node 10. Validated that all downstream nodes on that radial branch (Nodes 11-17) successfully reported as strictly de-energized.
*   **Failure 2 (Lateral Branch - Node 24):** Isolated Node 24, disconnecting its specific sub-branch.
*   **LV Configuration A (Default):** Static, direct mapping of synthetic customers to their nearest physical node.
*   **LV Configuration B (Hidden Reconfiguration):** Shifted synthetic customer mappings between adjacent nodes (Node 24 to Node 25) to mimic daily load balancing. The simulation successfully altered the interrupted-customer list while keeping the physical switch state hidden from the final feature output.

## 4. MV SCADA Measurements Validation
We extracted targeted state variables (Voltage Magnitude `vm_pu` converted to physical kV, and Line Current) strictly from assumed sensor locations (e.g., Feeder Head, Mid-feeder). 

**Findings:**
*   **Clear Fault Signatures:** Isolating a node creates highly detectable step-changes (`Delta_kV`) in upstream nodes. For example, a fault at Node 10 created a +0.338 kV upstream voltage bounce due to massive downstream load drop.
*   **Physical Validity:** De-energized islands correctly register as unavailable (`NaN` / `De-energized`), ensuring the ML model receives an accurate representation of a dead sensor rather than an invented mathematical voltage.

## 5. Stochastic Customer Call Simulation
We decoupled the ground-truth physical connectivity from the operator-visible outage reports by implementing a probabilistic behavioral model:
*   **Synthetic Customers:** Assigned populations of 10-30 synthetic customers to each LV branch/node.
*   **Stochastic Reporting:** Applied a Bernoulli distribution (`p_report = 0.40`) to true interrupted customers so that only a fraction actually generate a call.
*   **Observation Cutoffs:** Generated randomized reporting delays (2 to 45 mins) and successfully applied a simulated cutoff window (e.g., 15 mins).
*   **Outcome:** The final wide dataset aggregates `calls_total` and `calls_unique_customers` safely separated from `y_failed_component`, preventing data leakage.

## 6. Feasibility Gates Assessment
| Gate | Criteria | Status | Notes |
| :--- | :--- | :---: | :--- |
| **Network** | Express multiple LV switch states and identify disconnected customers safely. | ✅ Pass | `pandapower` correctly identifies de-energized islands; synthetic LV mapping works. |
| **Measurement** | MV SCADA produces varying signatures without hidden quantities. | ✅ Pass | `Delta_kV` proves upstream sensors register downstream load-drops distinctly. |
| **Information** | Competing regions produce measurably different evidence. | ✅ Pass | Combining geographic call clusters with electrical deltas successfully breaks ambiguities. |
| **Data** | Capable of reproducible generation on available hardware. | ✅ Pass | Extraction scripts execute in milliseconds per event; highly scalable. |
| **Evaluation** | Can generate strict, leakage-free data partitions. | ✅ Pass | Implementation of `cutoff_min` and hidden LV mapping ensures no future/ground-truth leakage. |

## 7. Next Steps & Target Resolution
Based on the PoC, the combined SCADA and call evidence is sufficient to distinguish broad service branches (e.g., "Branch 10-to-17"). Our supervised learning target (`y_region`) will be defined at the **service region/branch level** rather than the exact LV cable level, aligning perfectly with the project's real-world observability assumptions.

The next increment (`v0.0.4-design`) will formally define the project requirements, dependencies, and risk register based on this successful technical demonstration.