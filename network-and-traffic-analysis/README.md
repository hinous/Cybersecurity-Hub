# Cybersecurity Incident Analysis: Brute Force & Malware Redirection 🛡️

## 📝 Scenario Overview
This repository documents the investigation of a security breach at `yummyrecipesforme.com`. A disgruntled former employee executed a **brute force attack** to compromise the administrative panel. After gaining unauthorized access, the attacker injected malicious **JavaScript** code into the website's source code to redirect visitors and force malware downloads.

## 🔍 Technical Diagnosis (tcpdump Analysis)
By analyzing the network traffic logs (`tcpdump`), the following sequence of events was identified:

* **Initial Connection**: A standard DNS request and TCP handshake were established for the legitimate site.
* **Malicious Redirection**: At `14:20`, a new DNS request was triggered for `greatrecipesforme.com`.
* **Traffic Rerouting**: Traffic was diverted to the attacker's IP address **192.0.2.17** via the **HTTP port (80)**.
* **Payload Delivery**: The browser initiated a download request for an executable file containing malware using the `HTTP: GET` method.

## 🛠️ Mitigation & Hardening Recommendations
To prevent future incidents, the following **OS Hardening** measures are recommended:

* **Multi-Factor Authentication (MFA)**: Implement MFA to ensure that even if a password is stolen or guessed, a second verification step is required.
* **Account Lockout Policy**: Configure the system to automatically lock accounts after 3 failed login attempts to thwart brute force bots.
* **Strong Password Policy**: Enforce a minimum of 8 characters, including uppercase letters, numbers, and special symbols.

## 📁 Repository Files
* [Security incident report template.pdf](Security%20incident%20report%20template.pdf) - Finalized incident documentation in PDF format.
* [Security incident report template.docx](Security%20incident%20report%20template.docx) - Editable version of the incident report.
* [tcpdump traffic log.pdf](tcpdump%20traffic%20log.pdf) - Raw network traffic data used for the investigation.
