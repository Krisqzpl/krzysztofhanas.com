<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# SOC Lab Deployment Summary

## Overview

The lab is deployed in Azure and provides an end-to-end telemetry path from a public nginx web sensor to Microsoft Sentinel.

```text
Internet
   |
   v
Azure Ubuntu VM + nginx
   |
   v
/var/log/nginx/access.log
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
```

The same VM also hosts the SOC blog and a self-hosted GitHub Actions runner used for automated deployment.

> Secrets, credentials, tenant/subscription identifiers, private keys, and other sensitive environment-specific values are intentionally excluded.

## Azure Components

The environment consists of:

- Dedicated Azure resource group
- Log Analytics Workspace with ingestion cost controls
- Microsoft Sentinel
- Ubuntu Server 24.04 LTS VM
- Network Security Group
- Azure Monitor Agent
- Data Collection Rule for nginx custom text logs
- Custom Log Analytics table `NginxAccess_CL`

Public web traffic is allowed on TCP/80 and TCP/443. SSH access is restricted to the administrator's current public IP rather than exposed globally.

## Web Layer

nginx handles three public hostnames:

```text
krzysztofhanas.com
www.krzysztofhanas.com
blog.krzysztofhanas.com
```

The main website is served through nginx as a reverse proxy to Azure Static Web Apps.

The SOC blog is served locally from:

```text
/var/www/blog
```

HTTPS is provided with Let's Encrypt/Certbot and HTTP traffic is redirected to HTTPS.

## Security Telemetry

nginx uses a custom access-log format that includes the requested virtual host:

```nginx
log_format sentinel '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'host="$host"';
```

Telemetry is written to:

```text
/var/log/nginx/access.log
```

Azure Monitor Agent collects the log and a Data Collection Rule routes it to the Log Analytics custom table:

```text
NginxAccess_CL
```

Basic ingestion validation:

```kusto
NginxAccess_CL
| where TimeGenerated > ago(15m)
| project TimeGenerated, RawData
| order by TimeGenerated desc
```

The maintained parser is available at [../parsers/nginx-parser.kql](../parsers/nginx-parser.kql).

## Sentinel Detection Pipeline

KQL parsing extracts security-relevant fields such as:

```text
SourceIP | Host | Method | Path | Code | Bytes | Referer | UserAgent
```

Scheduled analytics rules use this normalized telemetry to identify suspicious activity and create Sentinel alerts/incidents.

Detection source files are maintained under [../detections/web/](../detections/web/).

## Blog CI/CD

The SOC blog uses GitHub as the source of truth.

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Self-hosted Runner (Ubuntu VM)
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

The workflow runs when content under `blog/**` changes on the `main` branch. The runner uses the custom `blog` label and runs as a system service.

Workflow: [../../.github/workflows/deploy-blog.yml](../../.github/workflows/deploy-blog.yml)

## Validation

A complete deployment is considered operational when:

1. Public websites are reachable over HTTPS.
2. nginx records requests with the target hostname.
3. Azure Monitor Agent collects the events.
4. The Data Collection Rule routes them to `NginxAccess_CL`.
5. KQL successfully parses the telemetry.
6. Sentinel analytics rules generate alerts/incidents when detection conditions are met.
7. Blog changes committed to GitHub are deployed automatically by the self-hosted runner.

## Security Considerations

Sensitive material such as SSH private keys, tokens, Azure credentials, passwords, API secrets, and private certificates must never be committed to the repository.

Screenshots and documentation should also be reviewed before publication for identifiers or infrastructure details that are not intentionally public.

For planned development, see [../FUTURE.md](../FUTURE.md).

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
