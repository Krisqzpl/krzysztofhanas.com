<p align="center">
  <a href="https://krzysztofhanas.com">
    <img src="../../assets/banner.png" alt="Krzysztof Hanas | Cybersecurity | SOC Analyst" width="60%">
  </a>
</p>

# CASE-003: UC010 - Path Traversal / Injection Probe

## Observation

After several days of monitoring, recurring suspicious web requests were observed in **UC010 - Path Traversal / Injection Probe**.

The activity appeared repeatedly from external sources and included multiple recognizable probing and exploitation patterns.

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

## Assessment

The observed requests are consistent with automated reconnaissance, path traversal attempts and generic vulnerability scanning.

No evidence of successful exploitation was identified from the reviewed activity.

The recurring patterns provide useful baseline data for further tuning of UC010 and help distinguish common Internet scanning noise from higher-risk activity.

---

<p align="center">
  © 2026 Krzysztof Hanas · <a href="https://krzysztofhanas.com">krzysztofhanas.com</a>
</p>
