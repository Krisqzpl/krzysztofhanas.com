<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# Microsoft Sentinel SOC Home Lab

A personal SOC lab for detection engineering, threat hunting, and incident investigation using Microsoft Sentinel and real Internet telemetry.

A public Ubuntu/nginx web server acts as the current telemetry source. nginx access logs are collected by Azure Monitor Agent, ingested into Log Analytics, parsed with KQL, and analyzed by Microsoft Sentinel analytics rules.

## Architecture

```text
Internet
   |
   v
krzysztofhanas.com / blog.krzysztofhanas.com
   |
   v
Azure Ubuntu VM + nginx
   |
   v
Azure Monitor Agent
   |
   v
Data Collection Rule
   |
   v
Log Analytics (NginxAccess_CL)
   |
   v
Microsoft Sentinel
   |
   +--> KQL Hunting
   +--> Analytics Rules --> Alerts / Incidents
```

Detailed architecture: [docs/architecture.md](docs/architecture.md)

## Telemetry

The primary source is nginx `access.log`. A custom log format includes the virtual host, allowing traffic to the main website and SOC blog to be distinguished.

Events are stored in the custom table:

```text
NginxAccess_CL
```

The KQL parser extracts fields including:

```text
SourceIP | Host | Method | Path | Code | Bytes | Referer | UserAgent
```

Parser: [parsers/nginx-parser.kql](parsers/nginx-parser.kql)

## Detection Engineering

The lab uses scheduled Sentinel analytics rules to identify suspicious web activity and create investigation-ready incidents.

Current detection scenarios include:

- **UC001** — Web Scanner Detection
- **UC002** — Sensitive File Discovery
- **UC003** — Potential Sensitive File Exposure
- **UC004** — High Request Rate

### Current tuning state

After the first 24-hour validation period, UC001–UC004 were temporarily paused while thresholds were tuned against observed Internet traffic. UC001, UC002 and UC004 generated 28 incidents during that period.

| Rule | Tuned logic | Window |
|---|---|---|
| **UC001** | More than 100 requests AND more than 75 unique paths for HTTP 404 activity | 1 minute |
| **UC002** | At least 75 unique sensitive paths | 5 minutes |
| **UC003** | Unchanged: any sensitive-path response with HTTP 2xx | 5-minute scheduled lookup |
| **UC004** | More than 100 requests AND more than 20 unique paths | 1 minute |

The thresholds were selected from a 7-day traffic baseline to reduce automated scanner noise while preserving higher-value detections.

Detection queries: [detections/web/](detections/web/)

## Real Internet Investigations

Because the web sensor is publicly reachable, it receives unsolicited Internet traffic such as automated enumeration of `.env`, `.git`, credential, configuration, and backup paths.

Observed activity is investigated in Sentinel and documented as cases when useful for the project.

Cases: [cases/](cases/)

## CI/CD

The SOC blog is deployed from GitHub through GitHub Actions and a self-hosted runner on the web server:

```text
GitHub --> GitHub Actions --> Self-hosted Runner --> rsync --> nginx
```

## Documentation

- [Architecture](docs/architecture.md)
- [Deployment summary](docs/setup.md)
- [Future development](FUTURE.md)
- [Detection queries](detections/web/)
- [Investigation cases](cases/)

## Project Focus

This lab is designed as a practical environment for developing and demonstrating:

**Microsoft Sentinel · KQL · Detection Engineering · Threat Hunting · Alert Triage · Incident Investigation · Detection Tuning · Security Automation**

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
