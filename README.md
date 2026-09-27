# Active Directory Cybersecurity Lab — Attack & Investigation

## Overview

This project is a small virtualised cybersecurity lab designed to demonstrate both offensive and defensive security concepts in an Active Directory environment.

The objective was to simulate activity from an attacker's perspective, then switch to the defender's perspective and investigate the resulting security telemetry.

The project followed the general workflow:

**Reconnaissance → Enumeration → Authentication Attempt → Detection → Investigation**

---

## Lab Architecture

The lab consisted of two virtual machines:

| System | Role | IP Address |
|---|---|---|
| DC01 | Windows Server / Active Directory Domain Controller | 192.168.56.101 |
| Kali Linux | Security testing / reconnaissance | 192.168.56.102 |

The virtual machines communicated over an isolated VirtualBox host-only network.

Kali Linux also had separate NAT connectivity for internet access when required.

### Domain

`cyberlab.local`

---

## Objectives

The main objectives of the lab were to:

- Build a basic Active Directory environment
- Configure users and groups
- Establish an isolated network between the virtual machines
- Perform reconnaissance against the domain controller
- Enumerate Windows/SMB services
- Simulate an authentication attempt against SMB
- Investigate the resulting Windows security event
- Understand the relationship between offensive activity and defensive telemetry

---

## 1. Environment Setup

The lab was built using VirtualBox on a Windows 11 host.

The host system had limited resources, so system resources were monitored throughout the exercise to ensure that both virtual machines could operate reliably.

The virtual machine images were verified using **SHA-256 hashes** before being used.

This provided confidence that the downloaded files matched their expected hashes and had not been altered or corrupted.

> Note: hash verification confirms file integrity against an expected hash; it does not by itself prove that a file is malware-free.

---

## 2. Active Directory Configuration

A Windows Server virtual machine was configured as the domain controller (`DC01`).

Active Directory Domain Services were configured for:

`cyberlab.local`

An Active Directory user and group were also created to provide a basic identity and access environment for testing.

The domain controller provided services including:

- Active Directory
- DNS
- Kerberos authentication
- LDAP
- SMB

---

## 3. Network Configuration

The Windows Server and Kali Linux virtual machines were connected through an isolated VirtualBox host-only network.

The lab used the following addressing:

    Network: 192.168.56.0/24

    DC01:  192.168.56.101
    Kali:  192.168.56.102

The isolated network allowed security testing to be performed without exposing the lab environment directly to the wider network.

Connectivity between the systems was first verified using `ping`.

---

## 4. Reconnaissance

Kali Linux was used to perform network reconnaissance against the domain controller.

### Nmap

The following command was used:

    nmap -sV 192.168.56.101

The `-sV` option was used to identify services and their versions where possible.

The scan identified a number of services associated with a Windows Active Directory environment, including:

| Port | Service |
|---|---|
| 53 | DNS |
| 88 | Kerberos |
| 135 | MSRPC |
| 139 | NetBIOS/SMB |
| 389 | LDAP |
| 445 | SMB |
| 464 | Kerberos password |
| 636 | LDAPS |
| 3268 | Global Catalog |
| 5985 | WinRM |

This provided an initial view of the domain controller's exposed attack surface.

---

## 5. Windows / SMB Enumeration

After identifying Windows-related services, `enum4linux` was used to perform more specialised Windows/SMB enumeration.

The purpose was to determine what information could be obtained about the Windows environment through SMB.

The domain was identified as:

    cyberlab.local

### Nmap vs enum4linux

The two tools served different purposes:

**Nmap**

> What services and ports are exposed?

**enum4linux**

> What information can I learn about the Windows/SMB environment?

This demonstrated the progression from broad network reconnaissance to more targeted enumeration.

---

## 6. SMB Authentication Testing

SMB was identified as an exposed service on TCP port 445.

A controlled authentication attempt was made against the SMB service using intentionally incorrect credentials.

The authentication attempt failed.

The purpose was not to gain access to the domain controller, but to generate realistic security telemetry that could then be investigated from the defensive side.

---

## 7. Defensive Investigation

After the failed authentication attempt, the perspective was switched from attacker to defender.

The Windows Security logs on the domain controller were examined to identify the resulting authentication event.

The event provided information including:

- The time of the event
- The source IP address
- The type of authentication activity
- Information about the failed login

This demonstrated the relationship between an action performed by an attacker and the telemetry generated on the target system.

The investigation followed a simplified SOC workflow:

    Attacker Activity
           ↓
    Authentication Attempt
           ↓
    Windows Security Event
           ↓
    Investigation
           ↓
    Determine Source / Context

---

## 8. Key Findings

### Finding 1 — Reconnaissance revealed the attack surface

Nmap identified multiple services exposed by the domain controller, including DNS, Kerberos, LDAP and SMB.

### Finding 2 — SMB provided additional enumeration opportunities

Once SMB was identified, specialised Windows/SMB enumeration provided additional information about the domain environment.

### Finding 3 — Authentication activity generated useful telemetry

The failed SMB authentication attempt resulted in a corresponding Windows Security event.

This demonstrated how an attacker's activity can become visible through endpoint and authentication logs.

### Finding 4 — Offensive and defensive perspectives are closely connected

Understanding how an attack is performed makes it easier to understand what defenders should look for in logs and security monitoring systems.

---

## 9. Lessons Learned

### 1. Reconnaissance is foundational

Before attempting an attack, understanding the available services and attack surface is critical.

### 2. Different tools answer different questions

Nmap provided broad network and service reconnaissance, while enum4linux provided more specialised Windows/SMB enumeration.

### 3. Authentication events are valuable security telemetry

A failed authentication attempt may appear insignificant by itself, but repeated or unusual authentication activity can provide an important indicator during an investigation.

### 4. Logs provide context for investigations

Security events can provide useful information such as timestamps, source addresses and authentication details that allow analysts to reconstruct activity.

### 5. Resource management matters in security labs

Because the lab was running on a system with limited RAM, Task Manager was used throughout the exercise to monitor system resources and troubleshoot performance issues.

---

## 10. Future Improvements

Potential extensions to the lab include:

- Add additional domain-joined Windows machines
- Create additional users with different privilege levels
- Simulate additional attack techniques
- Introduce Sysmon for richer endpoint telemetry
- Deploy a security monitoring platform such as Wazuh
- Develop detection rules for suspicious authentication activity
- Investigate lateral movement scenarios

---

## Conclusion

This project provided practical experience across both offensive and defensive cybersecurity concepts.

The lab progressed from:

**Building the environment → Reconnaissance → Enumeration → Authentication Testing → Detection → Investigation**

The most valuable aspect was being able to see the connection between an action performed from the attacker's perspective and the security telemetry generated on the target system.

This provided a practical foundation for further work in areas such as incident response, detection and response, threat hunting and security monitoring.
