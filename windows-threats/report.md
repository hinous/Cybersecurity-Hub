# Windows Threat Detection & DFIR Analysis

This directory is part of the Cybersecurity-Hub repository and contains detailed technical analysis, reconnaissance commands, detection matrices, and telemetry artifacts collected during Windows DFIR investigations.

---

## Directory Structure

```text
Cybersecurity-Hub/
└── Windows-threats/
    ├── README.md
    └── img/
        ├── rdp-bruteforce.png
        ├── logon-type3.png
        ├── 4624-logon-type10.png
        ├── download-event.png
        ├── extraction-event.png
        ├── process-create.png
        ├── lnk-shortcut.png
        ├── Double-extension.png
        ├── ms-defender.png
        ├── dns-2.png
        ├── ps-get-clipboard.png
        ├── xcopy-pdf.png
        ├── xcopy-xlsx.png
        ├── make-dir.png
        └── dns-stealer.pngExecutive Summary
A multi-vector attack chain on a Windows infrastructure was analyzed and mapped to the MITRE ATT&CK framework tactics:

Initial Access (TA0001):

Perimeter intrusion via RDP Brute Force (Event ID 4625 / 4624).

Social Engineering / Phishing vectors: ZIP downloads containing malicious shortcuts (.LNK) and binaries with double extensions (.jpg.exe).

BadUSB simulation / risk (automatic execution of HID scripts / keyboard payloads).

Defense Evasion (TA0005):

File extension masquerading.

Fileless execution using LOLBins (powershell -WindowStyle hidden).

EDR / Microsoft Defender agent verification and reconnaissance.

Discovery (TA0007):

Local system, group, user, and clipboard enumeration.

Collection (TA0009):

Document extraction (.pdf, .xlsx) and compressed staging (staging_58f1.zip).

Exfiltration (TA0010):

Covert data exfiltration channel to cloud infrastructure (AWS S3) via DNS requests (Event ID 22).

1. Initial Access and Defense Evasion Vectors
A. RDP Brute Force and Network Authentication
Technical Explanation and Detection
The Remote Desktop Protocol (RDP - Port 3389) is one of the most heavily exploited perimeter entry points. Attackers perform high-frequency credential stuffing and brute-force attacks against administrative accounts to gain unauthorized access.

Below is the chronological analysis of the security events recorded during the initial intrusion phase, detailing the failed brute-force attempts followed by successful administrative sessions and network authentications.

Brute Force Attack (Event ID 4625 - Failed Logon):

Date/Time: 5/20/2025 between 9:59:46 AM and 9:59:54 AM.

Recorded Events: 1,567 failed login attempts within a narrow timeframe.

TargetUserName: ADMINISTRATOR

Status Code: 0xC000006D (Incorrect username or bad password).

Successful RDP Logon (Event ID 4624 - LogonType 10):

Date/Time: 5/20/2025 9:51:49 AM

LogonType: 10 (RemoteInteractive - Remote interactive logon session).

Source Process: C:\Windows\System32\svchost.exe

Attacker Source IP: 203.205.34.107

NTLM V2 Network Authentication (Event ID 4624 - LogonType 3):

Date/Time: 5/20/2025 9:51:47 AM

LogonType: 3 (Network Logon via NTLM V2 / NtLmSsp).

Source Workstation: DESKTOP-QNBC4UU
