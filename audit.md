<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# SOC Lab Audit Log

A concise chronological record of changes made to the lab. Each row represents one implemented, modified, replaced, or retired element.

| # | Change | Component | Notes |
|---:|---|---|---|
| 32 | Added UC006–UC008 canary detection rules | Detection Engineering | Added GitHub-origin canary, blog-only canary and canary asset access detections with redacted public paths |
| 31 | Added VirusTotal and AbuseIPDB enrichment | Security Automation | Added IP reputation enrichment to support Sentinel investigations and incident context |
| 30 | Paused and tuned UC001–UC004 after noise review | Detection Tuning | After 24h of testing, UC001–UC004 were paused for tuning. 28 incidents were observed from UC001/UC002/UC004; new baseline-driven thresholds: UC001 >100 requests AND >75 unique paths / 1m, UC002 >=75 unique sensitive paths / 5m, UC003 unchanged, UC004 >100 requests AND >20 unique paths / 1m |
| 29 | Added IP reputation enrichment Logic App | Security Automation | Added incident-triggered enrichment using VirusTotal and AbuseIPDB, with results written back to Sentinel incident comments |
| 28 | Added known web scanners watchlist | Detection Tuning | Implemented `_GetWatchlist('WL_Known_Web_Scanners')` to reduce recurring Low-severity scanner alerts while preserving higher-risk detections |
| 27 | Aligned web detection severities with Sentinel | Detection Tuning | UC001 and UC002 set to Low; UC003 remains Medium; UC004 set to Low |
| 26 | Renamed existing web detections to UC001–UC004 | Detection Engineering | Standardized rule and file naming for current Network/Web use cases |
| 25 | Introduced structured Sentinel Use Case IDs | Detection Engineering | UC001–UC099 Network/Web, UC101–UC199 Identity, UC201–UC299 Endpoint/Host, UC301–UC399 Email, UC401–UC499 Cloud/Azure, UC501–UC599 Cross-source/Correlation |
| 24 | Connected portfolio website with SOC lab repository | Portfolio | Direct navigation to technical project content |
| 23 | Added GitHub Actions deployment | GitHub Actions | Automated deployment of project web content |
| 22 | Added SOC lab documentation and detection code to GitHub | GitHub | Portfolio and technical artifacts published |
| 21 | Began separating known scanner noise from security-relevant behavior | Detection Tuning | Known-scanner handling planned for noisy scan-only rules |
| 20 | Tuned detection logic based on observed scanner activity | Sentinel Analytics | Reduced noise and improved detection quality |
| 19 | Added Sensitive File Discovery detection | Sentinel Analytics | Detects probing for sensitive files/paths |
| 18 | Added High Request Rate detection | Sentinel Analytics | Detects request bursts from a source IP |
| 17 | Validated detection against unsolicited Internet traffic | Microsoft Sentinel | Confirmed rule behavior on real telemetry |
| 16 | Added entity mapping and custom alert details | Sentinel Analytics | Improved investigation context |
| 15 | Added Web Scanner Detection rule | Sentinel Analytics | Initial scanner detection |
| 14 | Created reusable KQL parser | KQL | Normalized Nginx telemetry for detections |
| 13 | Verified ingestion into `NginxAccess_CL` | Microsoft Sentinel | Custom log ingestion confirmed |
| 12 | Connected Nginx telemetry to Log Analytics | Log Analytics | Logs routed to the workspace |
| 11 | Created Data Collection Rule for Nginx logs | Azure Monitor | Defined log collection pipeline |
| 10 | Installed and configured Azure Monitor Agent | Azure Monitor | Log collection agent |
| 9 | Extended Nginx logging with requested host | Nginx | Added host context to telemetry |
| 8 | Connected custom domain and enabled HTTPS | Web / DNS | Public access configured |
| 7 | Installed and configured Nginx | Nginx | Public web service and telemetry source |
| 6 | Configured Network Security Group | Azure Networking | Web traffic allowed; SSH restricted |
| 5 | Deployed Ubuntu Server VM | Azure VM | Internet-facing web sensor |
| 4 | Added Log Analytics ingestion cost controls | Log Analytics | Limited lab ingestion costs |
| 3 | Enabled Microsoft Sentinel | Microsoft Sentinel | SIEM capabilities enabled for the workspace |
| 2 | Created Log Analytics Workspace | Azure / Log Analytics | Central workspace for lab telemetry |
| 1 | Created dedicated Azure Resource Group | Azure | Resource container for the SOC lab |
> New changes are appended as additional rows. Existing rows remain unchanged unless an implemented component is explicitly modified, replaced, or retired.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
