<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# SOC Lab Architecture

## Overview

The lab is built in Microsoft Azure around an Internet-facing Ubuntu VM running nginx. The server provides real web telemetry that is collected and analyzed in Microsoft Sentinel.

## Architecture

```text
Internet
   |
   v
Public Azure IP
   |
   v
Ubuntu VM
   |
   +--> nginx
   |      |
   |      v
   |   access.log
   |
   v
Azure Monitor Agent
   |
   v
Data Collection Rule
   |
   v
Log Analytics Workspace
   |
   v
NginxAccess_CL
   |
   v
Microsoft Sentinel
   |
   +--> KQL Hunting
   +--> Analytics Rules
   +--> Alerts / Incidents
```

## Main Components

**Azure VM**  
Ubuntu Server running nginx as the public web sensor.

**Network Security Group**  
Allows required web traffic on TCP/80 and TCP/443. SSH is restricted to a trusted source IP.

**nginx**  
Handles web traffic and records requests in `/var/log/nginx/access.log`.

**Azure Monitor Agent / DCR**  
Collects nginx access logs and forwards them to Log Analytics.

**Log Analytics / Sentinel**  
Stores telemetry in `NginxAccess_CL` and provides the platform for KQL analysis, detections, alerts, and investigations.

**GitHub**  
Stores documentation, KQL parsers, detection queries, and website content.

## Telemetry

The nginx log includes the source IP, HTTP request, response code, User-Agent, referrer, and requested host. A KQL parser converts the raw events into fields used by hunting and Analytics Rules.

## Purpose

The environment is intentionally small and public-facing so it can provide real Internet telemetry for SOC analysis without deploying intentionally vulnerable applications.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
