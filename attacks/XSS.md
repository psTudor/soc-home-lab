# XSS OWASP JuiceShop

---

## General Information

| Field | Details |
|---|---|
| Date |11.07.2026 |
| Tool used | Browser |
| OWASP Top 10 | A03:2021 - Injection  |
| MITRE ATT&CK | T1059.007 - JavaScript  |

---

## Brief Description

Cross-Site Scripting (XSS) allows an attacker to inject malicious 
JavaScript into a web page that is then executed in the victim's 
browser. Unlike SQL Injection which targets the database, XSS targets 
the end user. In a Reflected XSS attack, the payload is embedded in 
the URL and reflected back by the server without sanitization.

---

## Steps Taken

1. Accessed JuiceShop search bar via browser on Kali Linux
2. Entered classic XSS payload to test for filtering:
   `<script>alert('XSS')</script>`
3. Observed 500 Internal Server Error, payload caused SQLite error
4. Switched to alternative payload bypassing script tag restrictions:
   `<img src=x onerror=alert('XSS')>`
5. Payload was reflected in the page and executed by the browser
6. Alert popup confirmed successful XSS execution
7. Verified both payloads visible in Sentinel via NginxAccess_CL

---

## Attack Screenshots
![XSS Popup](/screenshots/XSS_search.png)
*Reflected XSS confirmed, alert popup triggered in browser via 
img onerror payload in JuiceShop search bar*

---

## Detection in Sentinel

### Detection Results

| Metric | Value | Description |
|---|---|---|
| Attack detected? | Yes | The attack was successfully identified via Nginx logs |
| Total XSS requests generated | 2 | Total HTTP requests containing script tags or encoded payloads sent against search/login fields |
| Malicious Events Logged | 2 | Requests containing visible XSS patterns in the URL parameters within the Nginx Logs |
| Detection Rate | 100% | Both malicious events logged in the SIEM were successfully detected by the KQL query |
| False positives | 0 | No legitimate traffic was flagged as malicious during the testing window |
| MTTD (Mean Time to Detect) | ~15 minutes | Time from attack execution to incident generation (based on the Scheduled Analytics Rule frequency) |

### Analytics Rule Configuration

| Setting | Value |
|---|---|
| Rule Type | Scheduled Analytics Rule |
| Rule Name | XSS Detection |
| Severity | High |
| Tactics (MITRE ATT&CK) | Execution, InitialAccess |
| Query frequency | 15 minutes |
| Query period | 15 minutes |
| Trigger threshold | greater than 1 |


### Sentinel Screenshots

![XSS Detection in Sentinel](/screenshots/XSS_alerts.png)
*Analytics Rule detected both XSS payloads. Script tag attempt 
and img onerror bypass visible in NginxAccess_CL*

---

## Conclusion

Sentinel successfully detected the XSS attack through the automated 
Analytics Rule configured before the attack. Both payloads were 
identified in NginxAccess_CL, the initial script tag attempt 
(which returned 500) and the successful img onerror bypass. 
Unlike the SQL Injection attack where detection was manual, 
the Analytics Rule generated an alert automatically within 8 minutes 
of the attack.

A potential improvement would be implementing a Content Security 
Policy (CSP) header on the Nginx proxy to block inline script 
execution, and a WAF rule to filter common XSS payload patterns 
before they reach the application.

