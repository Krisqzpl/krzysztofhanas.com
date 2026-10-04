<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# Microsoft Sentinel SOC Lab

Personal SOC lab built in Microsoft Azure for hands-on practice with Microsoft Sentinel, KQL, detection engineering, threat hunting, and incident investigation.

## Lab Overview

The lab uses a public Ubuntu/nginx web server as a telemetry source. Real Internet traffic is collected and sent to Microsoft Sentinel, where it is parsed with KQL and analyzed using custom Analytics Rules.

```text
Internet
   |
   v
Azure Ubuntu VM / nginx
   |
   v
Azure Monitor Agent
   |
   v
Log Analytics
   |
   v
Microsoft Sentinel
   |
   +--> KQL / Hunting
   +--> Analytics Rules
   +--> Alerts / Incidents
```

## Current Components

- Azure Ubuntu VM with nginx
- Network Security Group with restricted SSH access
- Azure Monitor Agent and Data Collection Rule
- Log Analytics Workspace
- Microsoft Sentinel
- Custom `NginxAccess_CL` telemetry
- KQL nginx parser
- Custom web detections
- Real Internet scanner telemetry
- GitHub-based project documentation and deployment

## Detection Work

The lab is used to build and tune detections based on observed web traffic, including:

- Web scanning and enumeration
- High request rates
- Sensitive file discovery
- Suspicious HTTP activity

Detection queries are stored under `detections/`.

## Investigations

Real activity observed by the sensor is used for investigation and documented as cases where useful.

## Documentation

- **Architecture** — short overview of the environment and telemetry flow
- **Audit Log** — chronological record of what was built and changed
- **Troubleshooting** — problems encountered and how they were resolved

## Goal

Build practical SOC experience by collecting real telemetry, writing detections, investigating alerts, and tuning rules based on observed activity.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
