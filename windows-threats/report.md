# Windows Threat Detection & DFIR Analysis

Welcome to the technical documentation of the `Cybersecurity-Hub` repository. This section walks you through a comprehensive DFIR (Digital Forensics and Incident Response) investigation of a multi-vector attack chain targeting Windows infrastructure. 

---

## Executive Summary

During our investigation, we mapped a sophisticated multi-vector intrusion against our Windows environment directly to the MITRE ATT&CK framework tactics. The attacker successfully gained an initial foothold via brute-force attacks on remote services, leveraged social engineering and malicious file extensions, bypassed endpoint defenses, and ultimately exfiltrated sensitive corporate data through covert DNS channels. 

---

## 1. Initial Access and Defense Evasion Vectors

### A. RDP Brute Force and Network Authentication

The attack begins at the perimeter. The Remote Desktop Protocol (RDP), running on port 3389, serves as the primary gateway exploited by the adversary. To gain unauthorized administrative entry, the attacker initiates a high-frequency credential stuffing and brute-force campaign against administrative accounts.

As we examine the Windows Security Event logs, the initial phase of this assault is clearly visible. The system records a massive wave of failed login attempts targeting the administrator account within a very tight time window:

* **Brute Force Attack (Event ID 4625 - Failed Logon):**
  * **Date/Time:** 5/20/2025 between 9:59:46 AM and 9:59:54 AM.
  * **Recorded Events:** 1,567 failed login attempts.
  * **TargetUserName:** ADMINISTRATOR
  * **Status Code:** `0xC000006D` (Incorrect username or bad password).

<p align="center">
  <a href="img/rdp-bruteforce.png" target="_blank">
    <img src="img/rdp-bruteforce.png" alt="RDP Brute Force 4625" width="600"/>
  </a>
</p>

Despite multiple security controls, persistent password guessing eventually yields results. Shortly after the brute-force flurry, the telemetry captures an active session establishment. The adversary successfully authenticates remotely via RDP, binding the session to a svchost process managed from an external attacker-controlled IP address:

* **Successful RDP Logon (Event ID 4624 - LogonType 10):**
  * **Date/Time:** 5/20/2025 9:51:49 AM
  * **LogonType:** 10 (RemoteInteractive - Remote interactive logon session).
  * **Source Process:** `C:\Windows\System32\svchost.exe`
  * **Attacker Source IP:** `203.205.34.107`

<p align="center">
  <a href="img/4624-logon-type10.png" target="_blank">
    <img src="img/4624-logon-type10.png" alt="RDP Successful Logon Type 10" width="600"/>
  </a>
</p>

Concurrently, network-level authentication logs reveal ancillary lateral movement and session handshakes using NTLM v2 protocols originating from an internal workstation within the network perimeter, confirming a broader compromised footprint:

* **NTLM V2 Network Authentication (Event ID 4624 - LogonType 3):**
  * **Date/Time:** 5/20/2025 9:51:47 AM
  * **LogonType:** 3 (Network Logon via NTLM V2 / NtLmSsp).
  * **Source Workstation:** `DESKTOP-QNBC4UU`

<p align="center">
  <a href="img/logon-type3.png" target="_blank">
    <img src="img/logon-type3.png" alt="NTLM Logon Type 3" width="600"/>
  </a>
</p>
