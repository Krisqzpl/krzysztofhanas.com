<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# SOC Lab — Future Work

A lightweight backlog for planned improvements and ideas. Items move into the audit log once implemented.

| # | Idea / Planned Change | Area | Status | Notes |
|---:|---|---|---|---|
| 3 | Create a known web scanners watchlist | Detection Tuning | Future | Suppress recurring scanner noise only in scan/noise-oriented detections |
| 4 | Apply known-scanner exclusions selectively | Detection Tuning | Future | Do not suppress security-relevant successful access or higher-risk behavior |
| 5 | Add Identity telemetry | Identity | Future | Use sign-in telemetry as the basis for UC101–UC199 detections |
| 6 | Add Endpoint / Host telemetry | Endpoint | Future | Basis for UC201–UC299 detections |
| 7 | Add Email telemetry and detections | Email | Future | Basis for UC301–UC399 detections |
| 8 | Add Azure / Cloud-focused detections | Cloud | Future | Basis for UC401–UC499 detections |
| 9 | Build cross-source correlation use cases | Correlation | Future | Correlate signals across Network, Identity, Endpoint, Email, or Cloud |
| 10 | Add Sentinel Workbook / dashboard | Visualization | Future | Summarize lab telemetry and detection activity |
| 11 | Expand threat-hunting scenarios | Threat Hunting | Future | Add reusable hunting queries and investigation examples |
| 12 | Maintain a dedicated troubleshooting document | Documentation | Future | Record only real issues, root causes, and resolutions encountered during the build |

## Use Case ID Ranges

| Range | Area |
|---|---|
| UC001–UC099 | Network / Web |
| UC101–UC199 | Identity |
| UC201–UC299 | Endpoint / Host |
| UC301–UC399 | Email |
| UC401–UC499 | Cloud / Azure |
| UC501–UC599 | Cross-source / Correlation |

## Status

- **Future** — accepted idea for a later stage.
- **Done** — implemented; the change should also be recorded in the audit log.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
