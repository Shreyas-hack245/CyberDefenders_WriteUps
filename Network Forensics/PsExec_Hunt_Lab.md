# CyberDefenders – PsExec Hunt Lab

**Category:** Network Forensics  
**Difficulty:** Easy  
**Tools Used:** Wireshark  
**Focus:** SMB Traffic Analysis, PsExec, Windows Service Execution, Credential Analysis, Lateral Movement

## 1. Lab Overview

The PsExec Hunt Lab focuses on investigating lateral movement within a Windows network using Server Message Block (SMB) traffic analysis.

The attacker initially gains access to a machine inside the network and uses PsExec to remotely execute commands on other systems.

PsExec is a legitimate Microsoft Sysinternals administration utility that allows administrators to execute processes remotely. However, attackers can abuse it to move laterally between compromised machines using administrative credentials and SMB shares.

The objective of this investigation is to analyze a PCAP file, identify the source of the attack, determine the compromised systems, and understand how PsExec was used to establish remote execution.

## 2. Investigation and Methodology

### Q1. Identify the IP address of the machine from which the attacker initially gained access

I began by examining the SMB traffic in Wireshark to identify communication between internal network systems.

I inspected the source and destination IP addresses associated with SMB sessions and traced the initial machine involved in the attack.

The analysis identified the machine with the following IP address:

**Finding:**

```text
10.0.0.130
```

This was the initial machine from which the attacker gained access to the network.

### Q2. Identify the hostname of the first machine targeted for lateral movement

After identifying the initial compromised machine, I examined the SMB communication to determine the hostname of the first target.

SMB negotiation and session setup traffic can reveal information about the systems involved in network communication.

The investigation identified the first pivot target as:

**Finding:**

```text
Sales-PC
```

This machine was the first system the attacker targeted for lateral movement.

### Q3. Identify the username used by the attacker for authentication

To identify the account used during lateral movement, I inspected the SMB Session Setup and authentication-related packets.

SMB authentication traffic contains information about the account attempting to establish a session.

The investigation revealed the following username:

**Finding:**

```text
ssales
```

This was the account used by the attacker to authenticate to the target system.

### Q4. Identify the service executable installed on the target machine

PsExec works by creating and executing a temporary Windows service on the remote machine.

I examined the SMB traffic for file transfers and service installation activity.

The analysis revealed the executable associated with PsExec:

**Finding:**

```text
psexesvc.exe
```

This executable is responsible for facilitating remote process execution through the PsExec service.

### Q5. Identify the network share used to install the service

PsExec commonly uses Windows administrative shares to transfer its service executable to remote systems.

I inspected the SMB Tree Connect requests to identify the administrative share accessed during installation.

The investigation identified:

**Finding:**

```text
ADMIN$
```

The `ADMIN$` share provides remote access to the Windows system directory, typically `C:\Windows`.

PsExec uses this share to transfer its service executable to the remote machine.

### Q6. Identify the network share used for communication

In addition to transferring the service executable, PsExec requires an SMB communication channel to interact with the remote system.

I examined the SMB tree connections and identified the share used for named-pipe communication.

**Finding:**

```text
IPC$
```

The `IPC$` share supports inter-process communication over SMB and is commonly used for remote administration and named-pipe communication.

### Q7. Identify the hostname of the second machine targeted for lateral movement

After analyzing the first compromised target, I continued examining SMB traffic to identify additional systems involved in the attack.

I inspected subsequent SMB sessions and correlated the destination systems with their hostnames.

The investigation revealed another machine targeted by the attacker:

**Finding:**

```text
Marketing-PC
```

This was the second machine targeted for lateral movement within the network.

## 3. Attack Timeline

The investigation reveals the following sequence of events:

1. **Initial Compromise:** The attacker gains access to the internal machine with IP address `10.0.0.130`.
2. **Reconnaissance:** The attacker identifies other systems accessible through SMB.
3. **Authentication:** The attacker uses the account `ssales` to authenticate to the target system.
4. **Lateral Movement:** The attacker pivots to `Sales-PC`.
5. **Service Installation:** PsExec transfers `psexesvc.exe` through the `ADMIN$` administrative share.
6. **Remote Communication:** PsExec uses the `IPC$` share for SMB-based communication.
7. **Further Lateral Movement:** The attacker targets another machine named `Marketing-PC`.

## 4. Key Security Findings

- An internal machine was used as the initial point of compromise.
- SMB was abused to facilitate lateral movement.
- The attacker authenticated using the account `ssales`.
- PsExec was used to execute processes remotely.
- The `ADMIN$` administrative share was used to transfer the service executable.
- The `IPC$` share was used for communication.
- Multiple internal machines were targeted during the intrusion.

## 5. Recommended Mitigations

1. Restrict SMB access between systems using network segmentation.
2. Disable SMBv1 and use secure SMB configurations.
3. Monitor access to administrative shares such as `ADMIN$` and `IPC$`.
4. Audit Windows service creation events, particularly Event ID 7045.
5. Monitor process creation events, particularly Event ID 4688.
6. Enable detailed Windows authentication auditing.
7. Use unique administrative credentials and implement least-privilege access.
8. Deploy Microsoft Defender or an EDR solution to detect suspicious remote service creation.
9. Monitor unexpected executions of `psexesvc.exe`.
10. Investigate abnormal SMB sessions and repeated authentication attempts.

## 6. Conclusion

The PsExec Hunt investigation demonstrates how legitimate Windows administration tools can be abused for malicious lateral movement.

By analyzing SMB traffic, authentication exchanges, administrative share access, and service installation activity, it is possible to reconstruct how an attacker moves between internal systems.

The lab highlights the importance of monitoring SMB communication, controlling administrative privileges, and detecting unexpected remote service creation.

**Tool used:** Wireshark.
