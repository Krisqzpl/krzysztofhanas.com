# SOC Lab Setup Guide

## Overview

This document describes the deployment process for the Microsoft Sentinel SOC Home Lab.

The goal is to build the following telemetry pipeline:

```text
Internet
   |
   v
Azure Ubuntu VM
   |
   v
nginx
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
Log Analytics
   |
   v
Microsoft Sentinel
```

The environment also hosts a small SOC blog deployed automatically from GitHub.

> This guide intentionally excludes passwords, private keys, tokens, public IP addresses, tenant IDs, subscription IDs and other environment-specific secrets.

---

# 1. Create the Azure Resource Group

Create a dedicated resource group for the lab.

Example:

```text
rg-soc-lab
```

Keeping the lab in a separate resource group makes it easier to:

- track costs,
- identify lab resources,
- manage permissions,
- remove the environment later.

---

# 2. Create Log Analytics Workspace

Create a Log Analytics Workspace.

Example:

```text
law-soc-lab
```

This workspace will receive the telemetry collected from the lab.

For a small home lab, configure an appropriate daily ingestion limit to prevent unexpected telemetry costs.

---

# 3. Enable Microsoft Sentinel

Enable Microsoft Sentinel on the Log Analytics Workspace.

The workspace becomes the central location for:

```text
Logs
KQL queries
Analytics rules
Alerts
Incidents
Threat hunting
```

---

# 4. Configure Cost Controls

Create an Azure Budget for the lab subscription or resource group.

Configure notifications at several thresholds.

Example:

```text
50%
80%
100%
```

Important:

Azure Budget alerts provide notifications but do not automatically stop resources.

Also consider limiting Log Analytics ingestion for a small lab.

---

# 5. Create the Ubuntu VM

Deploy an Ubuntu Server VM.

The lab currently uses:

```text
Ubuntu Server 24.04 LTS
```

A small VM size is sufficient for:

```text
nginx
Azure Monitor Agent
GitHub Actions runner
static blog
```

The VM requires:

```text
Public IP
Virtual Network
Subnet
Network Security Group
OS disk
SSH access
```

---

# 6. Configure Network Security Group

Allow public web traffic:

```text
TCP/80
TCP/443
```

Do not expose SSH globally unless there is a specific reason.

Restrict:

```text
TCP/22
```

to the administrator's current public IP using a `/32` source rule.

Example:

```text
X.X.X.X/32
```

If the administrator's public IP changes, update the NSG rule.

---

# 7. Connect to the VM

Connect using SSH:

```bash
ssh azureuser@<PUBLIC-IP>
```

Do not store private SSH keys in the GitHub repository.

---

# 8. Install nginx

Update packages:

```bash
sudo apt update
```

Install nginx:

```bash
sudo apt install nginx
```

Check the service:

```bash
sudo systemctl status nginx
```

Validate configuration whenever nginx configuration is modified:

```bash
sudo nginx -t
```

Only reload nginx after validation succeeds:

```bash
sudo systemctl reload nginx
```

---

# 9. Configure DNS

Create DNS records pointing the web domains to the Azure VM.

The lab uses:

```text
krzysztofhanas.com
www.krzysztofhanas.com
blog.krzysztofhanas.com
```

Conceptually:

```text
krzysztofhanas.com       -> Azure VM
www.krzysztofhanas.com   -> Azure VM
blog.krzysztofhanas.com  -> Azure VM
```

Existing email-related DNS records should remain unchanged.

Verify DNS resolution before configuring HTTPS.

---

# 10. Configure Main Website Reverse Proxy

Create an nginx virtual host for:

```text
krzysztofhanas.com
www.krzysztofhanas.com
```

The main website is hosted by Azure Static Web Apps.

nginx acts as a reverse proxy:

```text
Internet
   |
   v
nginx
   |
   v
Azure Static Web App
```

The configuration requires the upstream hostname and appropriate proxy headers.

Example structure:

```nginx
location / {
    proxy_pass https://<STATIC-WEB-APP-HOST>;

    proxy_ssl_server_name on;
    proxy_ssl_name <STATIC-WEB-APP-HOST>;

    proxy_set_header Host <STATIC-WEB-APP-HOST>;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Validate:

```bash
sudo nginx -t
```

Then reload:

```bash
sudo systemctl reload nginx
```

---

# 11. Configure HTTPS

Install Certbot and the nginx integration.

Example:

```bash
sudo apt install certbot python3-certbot-nginx
```

Request certificates for the required domains.

The lab certificate covers:

```text
krzysztofhanas.com
www.krzysztofhanas.com
blog.krzysztofhanas.com
```

Configure HTTP requests to redirect to HTTPS.

Verify automatic certificate renewal is enabled.

---

# 12. Create the SOC Blog

Create the blog directory:

```bash
sudo mkdir -p /var/www/blog
```

Create a dedicated nginx virtual host for:

```text
blog.krzysztofhanas.com
```

Use:

```text
/var/www/blog
```

as the document root.

Example:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name blog.krzysztofhanas.com;

    root /var/www/blog;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # TLS configuration omitted from this public example.
}
```

Configure the HTTP virtual host to redirect to HTTPS.

---

# 13. Disable Default nginx Site

If the default nginx site conflicts with the configured virtual hosts, disable it.

Example:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Validate:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

---

# 14. Configure nginx Security Logging

The standard access log was extended with the requested virtual host.

Inside the nginx `http` block configure:

```nginx
log_format sentinel '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'host="$host"';

access_log /var/log/nginx/access.log sentinel;
```

Validate:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Generate traffic to both websites and inspect:

```bash
sudo tail -n 10 /var/log/nginx/access.log
```

Expected hostname information:

```text
host="krzysztofhanas.com"
```

or:

```text
host="blog.krzysztofhanas.com"
```

---

# 15. Configure Azure Monitor Agent

Install/configure Azure Monitor Agent for the Ubuntu VM.

Azure Monitor Agent is responsible for collecting the nginx telemetry from:

```text
/var/log/nginx/access.log
```

---

# 16. Create Data Collection Rule

Create a Data Collection Rule for custom text logs.

Configure the source:

```text
/var/log/nginx/access.log
```

Configure the destination:

```text
Log Analytics Workspace
```

The lab stores nginx events in:

```text
NginxAccess_CL
```

---

# 17. Verify Log Ingestion

Generate requests to the website.

Then query:

```kusto
NginxAccess_CL
| where TimeGenerated > ago(15m)
| project TimeGenerated, RawData
| order by TimeGenerated desc
```

Confirm that recent nginx requests appear.

Also verify that new events contain:

```text
host="..."
```

---

# 18. Parse nginx Logs with KQL

Extract security-relevant fields:

```kusto
NginxAccess_CL
| extend SourceIP = tostring(split(RawData, " ")[0])
| extend Request = tostring(split(RawData, '"')[1])
| extend CodeByte = trim(" ", tostring(split(RawData, '"')[2]))
| extend Code = toint(split(CodeByte, " ")[0])
| extend Bytes = tolong(split(CodeByte, " ")[1])
| extend Method = tostring(split(Request, " ")[0])
| extend Path = tostring(split(Request, " ")[1])
| extend Referer = tostring(split(RawData, '"')[3])
| extend UserAgent = tostring(split(RawData, '"')[5])
| extend Host = extract('host="([^"]+)"', 1, RawData)
```

The maintained parser is stored in:

```text
soc-lab/parsers/nginx-parser.kql
```

---

# 19. Create First Analytics Rule

Create:

```text
Web Scanner Detection - Nginx
```

Detection logic:

```text
HTTP status = 404
Requests > 10
Unique paths > 5
Window = 1 minute
Grouping = SourceIP + Host
```

Configure the scheduled query to run every five minutes over the previous five minutes.

Map:

```text
IP.Address -> SourceIP
```

Add Custom Details:

```text
Host
Requests
UniquePaths
Paths
UserAgents
```

Enable incident creation.

The maintained detection source is stored in:

```text
soc-lab/detections/web/web-scanner.kql
```

---

# 20. Install GitHub Actions Self-Hosted Runner

Install the GitHub Actions runner on the Ubuntu VM.

Register it with the repository.

Add a custom label:

```text
blog
```

Install it as a service so it starts automatically.

Verify that the runner reports:

```text
Connected to GitHub
Listening for Jobs
```

---

# 21. Prepare Blog Deployment Permissions

The GitHub runner needs permission to update:

```text
/var/www/blog
```

Set appropriate ownership for the deployment user.

Example:

```bash
sudo chown -R azureuser:azureuser /var/www/blog
```

Do not give unnecessary broad permissions such as `777`.

---

# 22. Configure Blog CI/CD

Create:

```text
.github/workflows/deploy-blog.yml
```

The workflow triggers when files under:

```text
blog/**
```

change on the `main` branch.

Deployment concept:

```yaml
name: Deploy Blog

on:
  push:
    branches:
      - main
    paths:
      - "blog/**"

jobs:
  deploy:
    runs-on: [self-hosted, blog]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy blog to nginx
        run: |
          rsync -av --delete ./blog/ /var/www/blog/
```

---

# 23. Test CI/CD

Modify:

```text
blog/index.html
```

Commit the change.

Verify:

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Self-hosted Runner
   |
   v
/var/www/blog
   |
   v
Live Website
```

Do not manually edit the production blog after establishing GitHub as the source of truth.

---

# 24. Validate the Complete Lab

At this point verify the complete chain:

```text
Internet request
       |
       v
      DNS
       |
       v
     nginx
       |
       +------> Website
       |
       v
   access.log
       |
       v
      AMA
       |
       v
      DCR
       |
       v
 Log Analytics
       |
       v
NginxAccess_CL
       |
       v
    Sentinel
       |
       v
Analytics Rule
       |
       v
 Alert / Incident
```

A successful test should demonstrate:

1. Website is reachable over HTTPS.
2. nginx records the request.
3. Requested Host is recorded.
4. AMA collects the event.
5. DCR routes it to Log Analytics.
6. `NginxAccess_CL` receives it.
7. KQL parses the event.
8. Analytics rules can detect suspicious patterns.
9. Sentinel creates an alert/incident when detection conditions are met.

---

# Security Notes

Never commit:

```text
SSH private keys
GitHub tokens
Azure credentials
Passwords
API secrets
Private certificates
```

Before publishing screenshots, inspect them for:

```text
Public IP addresses
Tenant IDs
Subscription IDs
Workspace IDs
Usernames
Tokens
Session information
```

Public infrastructure details should only be published intentionally.

---

# Next Steps

The next planned detection is:

```text
Sensitive File / Secret Discovery
```

Examples include requests for:

```text
.env
.git/config
.aws/credentials
wp-config.php
```

Additional planned work includes website integrity monitoring and controlled ATT&CK simulations.
