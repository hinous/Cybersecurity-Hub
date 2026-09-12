  ## 3. Complete Indicator of Compromise (IoCs) Matrix

To conclude this forensic report, the following table summarizes all critical Indicators of Compromise (IoCs) identified throughout the investigation. These artifacts can be utilized by SOC analysts and threat hunters for retrospective log analysis, signature creation, and network defense hardening:

| Category | Indicator / Value | Description / Observation |
| :--- | :--- | :--- |
| **Affected Hosts** | `THM-PC`, `THM-DFIR-VM-2` | Target endpoints under forensic analysis |
| **Attacker IP** | `203.205.34.107` | External source originating successful RDP logins |
| **Source Workstation** | `DESKTOP-QNBC4UU` | Client host involved in NTLM v2 authentication |
| **C2 Domain (LNK)** | `http://wp16.hqywlqpa.thm:8000/cgi-bin/f` | Remote staging server hosting malicious PowerShell scripts |
| **Exfiltration Domain** | `collecteddata-storage-2025.s3.amazonaws.com` | AWS S3 cloud storage bucket receiving exfiltrated data |
| **Malicious Domain** | `exfil.beecz.cafe` | Unsuccessful DNS exfiltration test domain |
| **Malicious Archive** | `top-cats.zip` | Compressed archive downloaded via Microsoft Edge |
| **Malicious Shortcut** | `Official Website.lnk` | Shortcut file configured with hidden PowerShell execution |
| **Malicious Executable** | `best-cat.jpg.exe` | 8MB executable disguised as a JPEG image file |
| **Malicious Stealer** | `stealer.exe` | Compiled binary responsible for staging and exfiltration |
| **Staging Path** | `%Temp%\staging_58f1` | Temporary directory used for data aggregation |
| **Staging ZIP Path** | `%Temp%\staging_58f1.zip` | Final compressed file prepared for exfiltration |
