### v0.0.2-design: Related Work and Contribution

**Related Work Comparison:**
The fault localization problem in electrical distribution networks has been approached through various methodologies, ranging from traditional mathematical models to advanced machine learning and visual data fusion. The table below contrasts these existing methods against our proposed framework.

| Approach | Data Source | Topology Requirement | Limitations (Addressed by GP27) |
| :--- | :--- | :--- | :--- |
| **Analytical Impedance Method** (Hao et al., 2015) | Hardware disconnect switches and wiring fault indicators | Requires an exact, fully observable map of physical wire parameters and line lengths. | Fails entirely when the low-voltage (LV) grid connections are undocumented, unrecorded, or subject to daily switching changes. |
| **Digital Twin with Smart Meters** (DBSCAN & Random Forest) | Smart meter half-hourly telemetry and "last-gasp" voltage signals | Requires a complete LV network map to calculate the exact faulted branch. | Unusable for standard utilities without massive, expensive smart meter deployments. Completely ignores customer outage calls. |
| **Multimodal Visual Fusion** (Liu et al., 2026) | LiDAR, optical images, and physical GIS sensors | Strict Geographic Information System (GIS) mapping and physical line-of-sight spatial data. | Relies on highly expensive, unconventional visual sensors that are impractical for continuous, grid-wide utility deployment. |
| **Proposed Framework (GP27)** | Fused limited MV SCADA + Probabilistic Customer Calls | MV structure known; exact LV structure intentionally hidden and changing. | **None of the above.** Uses standard existing utility data and compensates for missing sensors/topology by combining pure electrical physics with stochastic human behavior. |

**Claimed Contribution:** 
Existing solutions demand either perfect mathematical observability, complete smart meter saturation, or expensive visual hardware. Our framework provides a reusable data-fusion architecture that successfully locates faults without requiring a complete LV single-line diagram or universal sensor coverage. By combining limited MV electrical measurements with stochastic, incomplete customer outage reports, the system ranks the most probable failure regions in messy, real-world conditions where traditional methods break down.