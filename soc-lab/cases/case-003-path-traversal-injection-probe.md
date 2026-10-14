<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# CASE-003: UC010 - Path Traversal / Injection Probe

## Observation

After several days of monitoring, recurring suspicious web requests were observed in **UC010 - Path Traversal / Injection Probe**.

The activity appeared repeatedly from external sources and included multiple recognizable probing and exploitation patterns.

<p align="center">
  <img src="https://github.com/user-attachments/assets/44feba2e-e28e-49c3-8c3e-9eede6367ded" width="31%" />
  <img src="https://github.com/user-attachments/assets/1cb1b24c-71b3-40de-8028-b665f37962dc" width="31%" />
  <img src="https://github.com/user-attachments/assets/0ebab9c5-8ceb-462a-bb5c-006770e15d96" width="31%" />
</p>

## Details

Examples of observed paths included:

    /admin/config?cmd=cat%20/root/%2eaws/credentials

Attempt to execute a command and read AWS credential files from the root user's home directory.

    /pms?module=logging&file_name=.../.aws/credentials

Suspicious file access pattern potentially targeting AWS credentials through path manipulation.

    /%2e%2e%5c.../var/log/apache2/access%2elog

Encoded path traversal attempt targeting the Apache access log.

    /[[%2e%2e%2ecatchAll]]

Unusual application route probing pattern, likely related to automated framework or endpoint discovery.

    /[[%2e%2e%2eoptional]]

Similar route probing behavior, likely used to identify exposed or incorrectly handled application routes.

## Apache Path Traversal / RCE Probe

Another recurring pattern was observed from multiple external IP addresses:

    /cgi-bin/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/bin/sh

The request uses heavily encoded traversal sequences in an attempt to escape the intended web directory and reach /bin/sh.

This structure is consistent with automated probing for the Apache HTTP Server path traversal vulnerabilities **CVE-2021-41773** and **CVE-2021-42013**. These vulnerabilities affected Apache HTTP Server 2.4.49 and 2.4.50 respectively. Under vulnerable configurations, path traversal could allow access to files outside the configured document root and, when CGI execution was enabled, could potentially lead to remote command execution.

The activity can therefore be interpreted as:

    Recon / Exploit Probe
            |
            v
    Encoded Path Traversal
            |
            v
    Filesystem Escape Attempt
            |
            v
    /bin/sh Target
            |
            v
    Potential Remote Code Execution

The same or highly similar payload being observed from multiple source IP addresses is consistent with commodity Internet scanning and automated exploitation attempts rather than normal web traffic.

The presence of the request alone does not indicate compromise. No evidence of successful exploitation was identified in the reviewed telemetry.

## Assessment

The observed requests are consistent with automated reconnaissance, path traversal attempts and generic vulnerability scanning.

No evidence of successful exploitation was identified from the reviewed activity.

The recurring patterns provide useful baseline data for further tuning of UC010 and help distinguish common Internet scanning noise from higher-risk activity.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
