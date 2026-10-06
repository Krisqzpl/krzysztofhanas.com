<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# CASE-002: Detection Tuning and Canary Deployment

## Summary

During the first ~48 hours of monitoring, Sentinel detected a high volume of automated web scanning against common administrative, backup and sensitive-file paths.

This exposed two things:
- UC001, UC002 and UC004 were too noisy for normal Internet-facing traffic.
- Pure threshold-based detections needed a higher-confidence detection layer.

## Tuning

| Rule | Change |
|---|---|
| UC001 | Raised request + unique path thresholds |
| UC002 | Raised unique sensitive-path threshold |
| UC004 | Required both high request volume and path diversity |

Goal: reduce routine Internet-scanning noise without losing aggressive reconnaissance.

## Canary Layer

Three additional detections were introduced:

- **UC006 - GitHub Canary Endpoint Access**  
  Detects interaction with an endpoint referenced only from a decoy resource in the public repository.

- **UC007 - Blog Canary Endpoint Access**  
  Detects access to a hidden endpoint not linked from legitimate website content.

- **UC008 - Canary Asset Access**  
  Detects access to an additional decoy resource used as a secondary reconnaissance signal.

Exact canary paths are intentionally excluded from public documentation.

## Detection Flow

    Internet traffic
          |
          v
    Nginx telemetry
          |
          v
    Microsoft Sentinel
          |
          +--> UC001 / UC002 / UC004
          |        |
          |        v
          |     tuning
          |
          +--> UC006 / UC007 / UC008
                   |
                   v
          higher-confidence signal

## Outcome

The lab now separates:
- routine Internet background scanning,
- aggressive enumeration,
- direct interaction with decoy resources.

**Result:** lower alert noise and better prioritization of suspicious reconnaissance.

## Key Lesson

**Collect telemetry → establish baseline → tune detections → identify gaps → add higher-confidence signals.**

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
