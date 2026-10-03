# SOC Lab Architecture

## Overview

This project is a personal Microsoft Sentinel SOC lab designed for practical work with real security telemetry.

The current environment uses a publicly accessible nginx server hosted in Microsoft Azure as an Internet-facing sensor. HTTP requests are collected and forwarded to Microsoft Sentinel, where they can be analyzed using KQL and converted into detection rules and incidents.

The architecture also hosts the project website and SOC blog, allowing the same environment to provide both security telemetry and project documentation.

---

## High-Level Architecture

```text
                         Internet
                            |
                            v
                  krzysztofhanas.com
               blog.krzysztofhanas.com
                            |
                            v
                     Public Azure IP
                            |
                            v
                  +-------------------+
                  | Azure Ubuntu VM   |
                  |                   |
                  |       nginx       |
                  +---------+---------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
        krzysztofhanas.com     blog.krzysztofhanas.com
                 |                     |
                 v                     v
          Reverse Proxy          Local Static Site
                 |               /var/www/blog
                 v
        Azure Static Web App

                            |
                            |
                  nginx access.log
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
                 +----------+----------+
                 |                     |
                 v                     v
            KQL Hunting         Analytics Rules
                                       |
                                       v
                                Alerts / Incidents
```

---

# Components

## Domain and DNS

The project uses:

```text
krzysztofhanas.com
```

Two web targets are currently monitored:

```text
krzysztofhanas.com
blog.krzysztofhanas.com
```

Both resolve to the public IP address of the Azure nginx server.

Existing email-related DNS records are kept separate from the SOC lab configuration and are not modified by the web sensor setup.

---

## Azure Ubuntu VM

The Internet-facing sensor runs on an Ubuntu Server virtual machine hosted in Microsoft Azure.

Its main responsibilities are:

- nginx web server
- reverse proxy
- SOC blog hosting
- nginx access logging
- Azure Monitor Agent
- GitHub Actions self-hosted runner

The VM is intentionally small because the project is designed as a low-cost home lab.

---

## Network Security

The Azure Network Security Group allows public web traffic on:

```text
TCP/80
TCP/443
```

SSH administration is restricted to the administrator's current public IP address using a `/32` rule.

This prevents SSH from being intentionally exposed to the entire Internet while keeping the web service publicly reachable.

---

# nginx Architecture

nginx performs two different roles.

## Main Website

Requests for:

```text
krzysztofhanas.com
www.krzysztofhanas.com
```

are reverse proxied to an Azure Static Web App.

Flow:

```text
Internet
   |
   v
nginx
   |
   v
Azure Static Web App
```

The public client therefore communicates with nginx while nginx communicates with the upstream website.

---

## SOC Blog

Requests for:

```text
blog.krzysztofhanas.com
```

are served directly from:

```text
/var/www/blog
```

Flow:

```text
Internet
   |
   v
nginx
   |
   v
/var/www/blog
```

The blog is maintained through GitHub rather than edited directly on the server.

---

# HTTPS

HTTPS certificates are provided by Let's Encrypt using Certbot.

The certificate covers:

```text
krzysztofhanas.com
www.krzysztofhanas.com
blog.krzysztofhanas.com
```

HTTP requests are redirected to HTTPS.

Certificate renewal is handled automatically by Certbot.

---

# GitHub and CI/CD

GitHub acts as the source of truth for website and project content.

The repository contains:

```text
Main website
SOC blog
SOC lab documentation
KQL parsers
Detection rules
Future scripts and hunting queries
```

The blog deployment pipeline is:

```text
Developer
    |
    v
GitHub Commit
    |
    v
GitHub Actions
    |
    v
Self-Hosted Runner
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

A GitHub Actions self-hosted runner is installed on the Azure VM.

Changes under:

```text
blog/**
```

trigger the blog deployment workflow.

This makes GitHub the authoritative source for blog content and avoids manual modification of production files.

---

# Security Telemetry Pipeline

The primary security telemetry source is the nginx access log:

```text
/var/log/nginx/access.log
```

The complete telemetry flow is:

```text
Internet Request
       |
       v
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
Log Analytics Workspace
       |
       v
NginxAccess_CL
       |
       v
Microsoft Sentinel
```

---

# nginx Log Format

The default nginx access log was extended to include the requested virtual host.

```nginx
log_format sentinel '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'host="$host"';

access_log /var/log/nginx/access.log sentinel;
```

Adding `$host` is important because multiple websites use the same nginx server and the same access log.

Without this field, a request such as:

```text
GET /.env
```

would not reliably indicate whether the scanner targeted:

```text
krzysztofhanas.com
```

or:

```text
blog.krzysztofhanas.com
```

The HTTP Referer cannot be used reliably for this purpose because automated scanners may omit or manipulate it.

---

# Azure Monitor Agent

Azure Monitor Agent collects:

```text
/var/log/nginx/access.log
```

from the Ubuntu VM.

The agent sends the data through an Azure Data Collection Rule.

No application-side integration with Microsoft Sentinel is required.

---

# Data Collection Rule

The Data Collection Rule defines:

- which log file is collected
- which machine provides the telemetry
- where the collected data is sent

The nginx log is sent to the project's Log Analytics Workspace.

---

# Log Analytics

nginx events are stored in the custom table:

```text
NginxAccess_CL
```

The original nginx event remains available as raw data.

KQL is then used to transform the raw log into security-relevant fields.

---

# KQL Normalization

The nginx parser extracts fields such as:

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

The parser source is stored in:

```text
soc-lab/parsers/nginx-parser.kql
```

Conceptually:

```text
RawData
   |
   v
KQL Parser
   |
   +--> SourceIP
   +--> Host
   +--> Method
   +--> Path
   +--> Code
   +--> Bytes
   +--> Referer
   +--> UserAgent
```

This creates a consistent set of fields for hunting and detection engineering.

---

# Microsoft Sentinel

Microsoft Sentinel provides the SIEM layer of the lab.

It is currently used for:

- log investigation
- KQL development
- threat hunting
- scheduled analytics
- alert generation
- entity mapping
- incident creation
- detection tuning

---

# Detection Layer

The first implemented analytics rule is:

```text
Web Scanner Detection - Nginx
```

It looks for a source IP generating:

```text
>10 HTTP 404 requests
>5 unique paths
within 1 minute
```

Activity is grouped by:

```text
SourceIP
Host
```

This means scanning against the main website and blog is evaluated separately.

The rule also exposes useful investigation context including:

```text
Host
Requests
UniquePaths
Paths
UserAgents
```

The complete rule is stored in:

```text
soc-lab/detections/web/web-scanner.kql
```

---

# Why Use a Public Web Sensor?

A public web server starts receiving unsolicited automated traffic relatively quickly.

This provides real examples of:

- web crawlers
- vulnerability scanners
- secret discovery attempts
- configuration discovery
- malformed requests
- automated reconnaissance

Examples observed in the lab include probes for:

```text
.env
.git/config
.aws/credentials
wp-config.php
```

The goal is not to intentionally expose vulnerable applications.

Instead, a minimal nginx service provides Internet telemetry that can be safely analyzed in the SIEM.

---

# Separation of Responsibilities

The project intentionally separates several functions.

```text
GitHub
    Source of truth for code and documentation

nginx
    Internet-facing web service and telemetry source

Azure Monitor Agent
    Log collection

Data Collection Rule
    Telemetry routing

Log Analytics
    Log storage and querying

Microsoft Sentinel
    Detection, investigation and incidents

SOC Blog
    Presentation and documentation of the project
```

This separation makes the environment easier to expand without turning the web server itself into the SIEM.

---

# Future Architecture

The lab is intended to grow over time.

Possible future telemetry sources include:

```text
Windows VM
Linux VM
Endpoint telemetry
Authentication logs
Controlled attack simulations
Website integrity telemetry
```

Future architecture may therefore evolve toward:

```text
                 Internet
                    |
                    v
              Web Sensor
                    |
                    |
      +-------------+-------------+
      |             |             |
      v             v             v
  Web Logs      Windows Logs   Attack Lab
      |             |             |
      +-------------+-------------+
                    |
                    v
             Log Analytics
                    |
                    v
            Microsoft Sentinel
                    |
          +---------+---------+
          |                   |
          v                   v
      Detection             Hunting
          |
          v
       Incidents
```

Controlled ATT&CK simulations may later be generated using tools such as Atomic Red Team or MITRE CALDERA.

---

# Design Principles

The lab follows several basic principles:

1. Keep the environment inexpensive.
2. Collect real telemetry where it can be done safely.
3. Do not intentionally deploy vulnerable production services.
4. Keep management interfaces restricted.
5. Store detection code and documentation in GitHub.
6. Separate telemetry collection from detection logic.
7. Document why each detection exists.
8. Build detections from observed behavior rather than only theoretical examples.
9. Expand the environment gradually.
10. Treat the lab as both a learning platform and a security engineering portfolio.
