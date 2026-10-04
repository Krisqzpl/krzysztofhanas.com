# SOC Lab Build Audit Log

A short chronological record of the main changes made while building the lab.

## Environment

1. Created a dedicated Azure Resource Group for the SOC lab.
2. Created a Log Analytics Workspace and enabled Microsoft Sentinel.
3. Added cost controls for Log Analytics ingestion.
4. Deployed an Ubuntu Server VM as the Internet-facing web sensor.
5. Configured the Network Security Group for TCP/80 and TCP/443 and restricted SSH access.
6. Installed and configured nginx.
7. Connected the domain and enabled HTTPS.
8. Added the project website and SOC blog infrastructure.

## Telemetry

9. Extended nginx logging to include the requested host.
10. Installed/configured Azure Monitor Agent.
11. Created a Data Collection Rule for nginx access logs.
12. Connected the log source to Log Analytics.
13. Verified ingestion into `NginxAccess_CL`.
14. Created a reusable KQL parser for nginx telemetry.

## Detection Engineering

15. Created the first Web Scanner Detection rule.
16. Added Sentinel entity mapping and custom alert details.
17. Validated detections against real unsolicited Internet traffic.
18. Added additional detection logic for high request rates and sensitive-file discovery.
19. Tuned detection logic based on observed scanner activity.
20. Started separating known scanner noise from security-relevant behavior.

## GitHub / Deployment

21. Added the SOC lab documentation and detection code to GitHub.
22. Added GitHub Actions-based deployment for project web content.
23. Connected the public portfolio website with the SOC lab repository.

## Current State

The lab currently collects real nginx telemetry, parses it with KQL, runs custom Sentinel Analytics Rules, and produces alerts/incidents for investigation.

Detailed technical problems and their resolutions will be maintained separately in `TROUBLESHOOTING.md`.
