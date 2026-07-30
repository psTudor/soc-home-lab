# Directory Traversal / Path Traversal

## General Information

| Field | Details |
|---|---|
| Date | 18.07.2026 |
| Tool used | ffuf (Fuzz Faster U Fool) |
| OWASP Top 10 | A01:2021 - Broken Access Control|
| MITRE ATT&CK | T1083 - File and Directory Discovery |

---

## Brief Description

A path traversal / directory traversal vulnerability allows an attacker to access files and directories that are stored outside the web root folder. By manipulating variables that reference files with (../) sequences and its variations, or by using absolute file paths, an attacker may be able to access arbitrary files and directories stored on the file system.

---

## Steps Taken

1. Identified the /ftp/ endpoint on OWASP Juice Shop as the target for file retrieval.
2. Executed manual path traversal payloads using curl against the Nginx reverse proxy (Port 80):

```bash 
curl -v "http://[VM-IP]/ftp/../../../../etc/passwd"
```
3. Automated the attack simulation using ffuf to fuzz the endpoint with a wordlist:
```bash
ffuf -u "http://[VM-IP]/ftp/FUZZ" -w /usr/share/wordlists/dirb/common.txt -fc 403,404
```

---

## Attack Screenshots

![fuff tool](/screenshots/traversal_ffuf.png)
*Attack automated with ffuf tool*
---

## Detection in Sentinel

### Detection Results

| Metric | Value | Description |
|---|---|---|
| Attack detected? | Yes | The attack was successfully identified via Nginx logs |
| Total traversal requests generated | 2 | Total HTTP requests sent via curl and ffuf fuzzing the /ftp/ endpoint |
| Malicious Events Logged | 2 | Requests containing visible path traversal patterns (../, %2e%2e) in the Nginx Logs |
| Detection Rate | 100% | Both malicious events logged in the SIEM were successfully detected by the KQL query |
| False positives | 0 | No legitimate traffic was flagged as malicious during the testing window |
| MTTD (Mean Time to Detect) | ~15 minutes | Time from attack execution to incident generation (based on the Scheduled Analytics Rule frequency) |

### Analytics Rule Configuration

| Setting | Value |
|---|---|
| Rule Type | Scheduled Analytics Rule |
| Rule Name | Path Traversal Detection |
| Severity | High |
| Tactics (MITRE ATT&CK) | Discovery, InitialAccess |
| Query frequency | 15 minutes |
| Query period | 15 minutes |
| Trigger threshold | greater than 1 |

### Sentinel Screenshots

![kql_traversal](/screenshots/traversal_kql.png)
*The KQL used to detect the attack*

---

## Conclusion

Sentinel successfully detected the path traversal attack by parsing Nginx logs for ../ and encoded variants within approximately 15 minutes. The custom KQL query identified the malicious file retrieval requests targeting the /ftp/ endpoint, which served as the primary detection indicator since HTTP access logs capture the requested URI. A potential improvement would be deploying a Web Application Firewall (WAF) like ModSecurity to actively block path traversal sequences in real time before they reach the backend application.


