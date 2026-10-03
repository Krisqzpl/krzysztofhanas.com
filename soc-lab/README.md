# Microsoft Sentinel SOC Home Lab

A personal Security Operations Center lab built to practice detection engineering, log analysis, threat hunting, and incident investigation using Microsoft Sentinel and real Internet telemetry.

The lab currently uses a public Ubuntu/nginx web server as a telemetry source. Internet traffic is collected by Azure Monitor Agent and forwarded to Microsoft Sentinel, where KQL queries and analytics rules are used to detect suspicious activity.

## Current Architecture

```text
                    Internet
                       |
                       v
             krzysztofhanas.com
          blog.krzysztofhanas.com
                       |
                       v
              Azure Ubuntu VM
                    nginx
                 /         \
                /           \
               v             v
       Main Website        SOC Blog
       Reverse Proxy     /var/www/blog
               \             /
                \           /
                 access.log
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
            Microsoft Sentinel
                     |
              +------+------+
              |             |
              v             v
          KQL Hunting   Analytics Rules
                            |
                            v
                     Alerts / Incidents
```

## What the Lab Currently Does

The environment currently provides:

- Public nginx web telemetry collection
- Microsoft Sentinel log ingestion
- Custom nginx log parsing with KQL
- Source IP extraction
- HTTP method and requested path extraction
- HTTP response code analysis
- User-Agent analysis
- Referrer analysis
- Virtual host identification
- Scheduled analytics rules
- Sentinel entity mapping
- Custom alert details
- Incident creation and grouping
- Real Internet scanner observation

## Data Source

The primary telemetry source is:

```text
/var/log/nginx/access.log
```

nginx uses a custom log format that includes the requested virtual host:

```nginx
log_format sentinel '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'host="$host"';
```

This allows Microsoft Sentinel to distinguish traffic targeting:

```text
krzysztofhanas.com
blog.krzysztofhanas.com
```

## Log Pipeline

```text
nginx
  |
  v
access.log
  |
  v
Azure Monitor Agent
  |
  v
Data Collection Rule
  |
  v
Log Analytics
  |
  v
NginxAccess_CL
  |
  v
Microsoft Sentinel
```

The raw nginx events are stored in the custom Log Analytics table:

```text
NginxAccess_CL
```

## KQL Parsing

Raw nginx events are parsed with KQL to extract useful security fields.

Example fields:

```text
SourceIP
Host
Method
Path
Code
Bytes
Referer
UserAgent
```

The parser is maintained separately under:

```text
parsers/nginx-parser.kql
```

## Detection Engineering

The first analytics rule implemented in the lab is:

### Web Scanner Detection

The rule identifies a source IP requesting many non-existing resources within a short period.

Current detection logic:

```text
HTTP response: 404
Requests: >10
Unique paths: >5
Time window: 1 minute
Aggregation: SourceIP + Host
```

The detection includes:

- IP entity mapping
- Target hostname
- Request count
- Unique path count
- Requested paths
- User-Agent information

The complete KQL detection is stored under:

```text
detections/web/web-scanner.kql
```

## Real Internet Telemetry

Because the nginx server is publicly reachable, the lab receives real unsolicited Internet traffic.

Observed activity has included automated attempts to discover files and resources such as:

```text
.env
.git/config
.aws/credentials
wp-config.php
backup/configuration files
```

This provides real telemetry for practicing investigation and detection engineering without exposing production infrastructure.

## CI/CD and SOC Blog

The project also includes a small SOC blog hosted at:

```text
blog.krzysztofhanas.com
```

Blog content is maintained in GitHub.

The deployment flow is:

```text
GitHub commit
      |
      v
GitHub Actions
      |
      v
Self-hosted Runner
      |
      v
rsync
      |
      v
/var/www/blog
      |
      v
nginx
```

This allows changes committed to the `blog/` directory to be automatically deployed to the web server.

## Planned Detections

Future detections include:

- Sensitive File / Secret Discovery
- Successful Suspicious Requests
- High Request Rate / Web Flood
- HTTP Method Anomalies
- Known Scanner User-Agents
- Protocol and malformed request detection
- Path Traversal
- Command Injection patterns
- SQL Injection patterns
- XSS patterns
- Cross-detection IP correlation

## Planned Lab Expansion

Future stages of the project include:

### Website Integrity Monitoring

Periodically calculate a SHA-256 hash of website content and send change telemetry to Sentinel.

```text
Website
   |
   v
SHA-256
   |
   v
Compare Previous Hash
   |
   v
Sentinel
   |
   v
Integrity Alert
```

### Controlled ATT&CK Simulation

Generate controlled security telemetry using tools such as:

- Atomic Red Team
- MITRE CALDERA

The goal is to reproduce known ATT&CK techniques and develop corresponding Sentinel detections.

## Repository Structure

```text
soc-lab/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   └── setup.md
│
├── parsers/
│   └── nginx-parser.kql
│
├── detections/
│   └── web/
│       └── web-scanner.kql
│
├── hunting/
├── scripts/
└── screenshots/
```

## Project Goals

The goal of this project is not only to deploy Microsoft Sentinel, but to build a practical environment for developing SOC skills:

- Detection engineering
- KQL development
- Threat hunting
- Alert triage
- Incident investigation
- Telemetry analysis
- Detection tuning
- Security automation
- ATT&CK-based testing

The lab will be expanded gradually as new telemetry sources and detection scenarios are introduced.
