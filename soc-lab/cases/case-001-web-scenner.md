# CASE-001: Automated Web Scanner and Sensitive File Discovery

## Case Summary

| Field | Value |
|---|---|
| Case ID | CASE-001 |
| Detection | Web Scanner Detection - Nginx |
| Severity | Medium |
| Source IP | 130.12.180.117 |
| Target | blog.krzysztofhanas.com / krzysztofhanas.com |
| Activity | Automated web scanning / sensitive file discovery |
| Status | Investigated |
| Assessment | True Positive - Automated Internet Scanning |
| Impact | No evidence of compromise |

## Detection

Microsoft Sentinel generated an alert after detecting a high number of HTTP 404 responses from a single source IP across multiple unique paths within a short time window.

The detection identified multiple requests from the same source, multiple unique requested paths and repeated HTTP 404 responses. The source IP was successfully mapped as an IP entity in Microsoft Sentinel.

## Observed Activity

The source generated multiple requests attempting to discover potentially sensitive files and configuration resources.

Examples included:

    /.env
    /.env.backup
    /.env.old
    /.env.save
    /.env.bak
    /.env.prod
    /.env.production
    /.env.staging
    /.env.local
    /.env.live
    /.env.dev
    /.env.stage
    /.git/HEAD
    /config/.env
    /app/.env
    /api/.env
    /application/.env
    /functions/.env

An encoded path traversal-style request was also observed:

    /%2e%2e%2f%2eenv

The activity was consistent with automated reconnaissance and sensitive-file discovery rather than normal web browsing.

## User-Agent Analysis

The observed User-Agent contained a reference to the Yokohama Institute of Information Security and the `iisec.ac.jp` domain.

The User-Agent was treated only as an investigation artifact and not as proof of attribution. HTTP User-Agent strings are controlled by the client and can be modified or spoofed.

## Reputation Analysis

External reputation sources showed previous negative reputation associated with the source IP.

During the investigation, VirusTotal showed detections from multiple security vendors and AbuseIPDB showed a high abuse confidence score.

These reputation results supported the assessment that the IP had previously been associated with suspicious Internet activity. Reputation data was treated as supporting context rather than standalone evidence of malicious intent.

## HTTP Response Analysis

The observed requests primarily resulted in:

    404 - Not Found
    301 - Redirect

HTTP 301 responses were associated with redirection behavior and were not considered evidence that a sensitive file had been successfully retrieved.

No successful retrieval of the probed sensitive resources was identified during this investigation.

A `200 OK` response to one of the sensitive paths would require additional investigation because it could indicate that a requested resource was accessible.

## Assessment

The detection was classified as:

**True Positive - Automated Internet Scanning**

The analytics rule correctly identified scanning behavior.

The activity included attempts to enumerate sensitive configuration files and resources, but no evidence of successful access to those resources or subsequent compromise was identified.

The activity may represent automated research, reconnaissance, vulnerability scanning or other Internet-wide scanning activity.

The identity or organization operating the scanner was not attributed based solely on the User-Agent.

## Impact

No evidence of compromise was identified.

Observed sensitive-file requests returned unsuccessful responses or redirects, and no evidence was found that sensitive configuration data was exposed.

## Detection Engineering Outcome

Investigation of this alert identified an additional detection opportunity.

The original `Web Scanner Detection - Nginx` rule detected the activity based primarily on request volume, unique paths and HTTP 404 responses.

However, the investigation showed that attempts to access resources such as `.env`, `.git`, `.aws` and `wp-config` are independently security-relevant.

This resulted in development of a second Microsoft Sentinel analytics rule:

**Sensitive File Discovery - Nginx**

The new detection identifies a source attempting to access multiple unique sensitive paths within a short time window.

This provides semantic detection of sensitive-resource discovery in addition to the original behavioral web-scanning detection.

## Lessons Learned

This case demonstrates that a correctly triggered security alert does not necessarily indicate a successful compromise.

It also demonstrates how investigation of real Internet telemetry can be used to improve detection coverage.

    Real Internet Traffic
            |
            v
    Web Scanner Alert
            |
            v
    SOC Investigation
            |
            v
    Sensitive File Discovery Identified
            |
            v
    New Detection Logic
            |
            v
    Sensitive File Discovery Analytics Rule

The case therefore resulted not only in investigation of an individual scanner, but also in an improvement to the detection capabilities of the SOC lab.
