# CyberDefenders – Tomcat Takeover Lab Writeup

**Category:** Network Forensics  
**Difficulty:** Easy  
**Tools Used:** Wireshark, NetworkMiner  
**Platform:** CyberDefenders  
**Status:** Completed – 8/8 Questions

## 1. Scenario

A suspicious activity alert was raised against a web server. The objective of this investigation was to analyze the provided PCAP file using Wireshark and NetworkMiner to identify potential reconnaissance, unauthorized administrative access, credential compromise, malicious file uploads, and reverse shell activity.

The investigation focused on identifying the attacker's IP address, understanding the attack sequence, and determining how the attacker gained access to the Tomcat server.

## 2. Investigation and Findings

### Q1. Identify the attacker's IP address

**Answer:** `14.0.0.120`

I examined the TCP conversations in Wireshark using:

`Statistics → Conversations → TCP`

I observed connections to multiple destination ports on the target server. The unusual variation and sequential pattern of destination ports indicated possible port scanning activity.

The source IP responsible for initiating these connections was identified as `14.0.0.120`.

### Q2. Identify the attacker's country

**Answer:** China

Using an IP geolocation lookup, I identified the geographical origin associated with the attacker's IP address as China.

### Q3. Identify the web server administration port

**Answer:** `8080`

I filtered HTTP traffic originating from the attacker's IP:

`ip.src == 14.0.0.120 && http`

By examining the destination ports of the HTTP requests, I identified port `8080` as the port hosting the Tomcat administration interface.

### Q4. Identify the directory enumeration tool

**Answer:** Gobuster

I examined HTTP requests and responses, particularly requests that resulted in `404 Not Found` responses.

By following the HTTP streams and inspecting the HTTP User-Agent header, I identified Gobuster as the tool used to enumerate directories and files on the web server.

### Q5. Identify the discovered administration directory

**Answer:** `/manager`

I filtered successful HTTP responses using:

`http && http.response.code == 200`

After reviewing the corresponding request URIs, I identified `/manager` as the discovered Tomcat administration directory.

### Q6. Identify the compromised credentials

**Answer:** `admin:tomcat`

I investigated HTTP authentication headers to identify the credentials used to access the administration interface.

The captured traffic revealed successful authentication using the username `admin` and password `tomcat`.

### Q7. Identify the malicious uploaded file

**Answer:** `JXQOZY.war`

I filtered HTTP POST requests originating from the attacker's IP:

`ip.src == 14.0.0.120 && http.request.method == "POST"`

By following the HTTP stream and inspecting the uploaded file data, I identified the malicious WAR file named `JXQOZY.war`.

The upload indicated an attempt to deploy a malicious application on the Tomcat server.

### Q8. Identify the reverse shell callback destination

**Answer:** `14.0.0.120:443`

I examined TCP SYN-ACK traffic using:

`ip.src == 14.0.0.120 && tcp.flags == 0x012`

By following the relevant TCP stream, I identified the reverse shell connection details. The payload contained a `/dev/tcp/` redirection that revealed the callback destination.

The compromised server attempted to establish a reverse shell connection to the attacker's machine at `14.0.0.120` on port `443`.

## 3. Attack Timeline

The investigation revealed the following attack sequence:

1. **Reconnaissance:** The attacker scanned multiple ports on the target server.
2. **Service Discovery:** The attacker identified the Tomcat administration service running on port 8080.
3. **Directory Enumeration:** Gobuster was used to discover hidden directories, including `/manager`.
4. **Credential Access:** The attacker successfully authenticated using `admin:tomcat`.
5. **Execution:** A malicious WAR file named `JXQOZY.war` was uploaded to the server.
6. **Command and Control:** The malicious payload attempted to establish a reverse shell connection to `14.0.0.120:443`.

## 4. Key Security Observations

- Exposing administrative interfaces to untrusted networks increases the attack surface.
- Default or weak credentials can allow unauthorized administrative access.
- Directory enumeration can reveal sensitive application endpoints.
- Unrestricted WAR deployment can allow attackers to execute malicious code on a Tomcat server.
- Outbound connections should be monitored to detect potential reverse shell activity.

## 5. Conclusion

This investigation demonstrated how network traffic analysis can reveal a complete web server compromise attempt, from initial reconnaissance to malicious application deployment and reverse shell communication.

Using Wireshark's display filters, TCP conversations, HTTP streams, and packet inspection, I was able to reconstruct the attack sequence and identify the attacker's infrastructure, enumeration tool, compromised credentials, malicious payload, and callback destination.

**Key Learning:** Network forensics is not just about examining individual packets. Correlating multiple network events helps reconstruct an attacker's actions and understand how a server was compromised.
