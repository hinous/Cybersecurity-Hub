# Ransomware Incident Response Playbook

### Step 1: Preparation (Preparacion)

The goal of this phase is to establish the technical and legal foundation required to respond to a ransomware incident without hesitating.

* Data Resiliency (3-2-1 Backup Rule):
  * Maintain three copies of all medical records.
  * Store them on two different media types (Cloud and Local Server).
  * Keep one copy Offline or Air-gapped to ensure the ransomware cannot reach the backups.

* Legal and Compliance Readiness:
  * Pre-define a Legal Counsel Task Force to evaluate the legality of ransom payments under local and international laws (Mexico LFPDPPP).
  * Establish direct contact channels with INAI (Mexico) and HHS/OCR (USA) for regulatory reporting.
  * Draft a Communications Plan for authorities and affected patients to meet mandatory notification deadlines.

* Access Control and Hardening:
  * Implement MFA (Multi-Factor Authentication) on all administrative accounts.
  * Enforce the Principle of Least Privilege (PoLP).

* Technical Inventory:
  * Maintain an updated list of all Critical Assets, including life-support systems, imaging servers (PACS), and Electronic Health Records (EHR).

---

### Step 2: Detection and Analysis (Deteccion y Analisis)

The focus of this phase is to confirm the ransomware infection, identify its point of entry, and assess the potential legal and clinical impact.

* SIEM Alerting (Splunk/Chronicle):
  * Monitor for high CPU usage and massive file renaming/deletion events on the EHR server.
  * Detect a surge in Access Denied logs, indicating unauthorized encryption attempts.

* Indicator of Compromise (IoC) Identification:
  * Search for known ransomware file extensions (.wannacry, .locked) and ransom notes.

* Clinical Impact Assessment:
  * Determine if the affected systems include Medical Imaging (PACS) or life-support monitoring databases.
  * Prioritize the analysis of servers containing Sensitive Personally Identifiable Information (SPII).

* Legal and Forensic Preservation:
  * Begin a Chain of Custody log to document all findings.
  * Calculate the Discovery Time to ensure compliance with INAI/HHS notification windows.

---

### Step 3: Containment (Contencion)

The primary objective is to stop the ransomware from spreading to other clinical areas and to quarantine the infection.

* Short-Term Containment (Technical Isolation):
  * Network Segmentation: Immediately disconnect infected VLANs from the clinical network.
  * Endpoint Isolation: Use EDR tools to isolate infected devices.

* Service Continuity (The Paper Protocol):
  * Activate the Hospital Contingency Plan: Switch to manual records to ensure patient care continues.
  * Divert emergency cases if critical diagnostic systems (CT Scans/MRI) are compromised.

* System Preservation (Legal Evidence):
  * Avoid Reformatting: Take a Memory Dump or Disk Image first for insurance and legal evidence.

* Financial Containment:
  * Freeze any emergency payments until the legal team confirms the policy regarding ransom demands.

---

### Step 4: Eradication (Erradicacion)

The goal of this phase is to eliminate all components of the ransomware and identify the Patient Zero.

* Malware Removal and Remediation:
  * Perform a full system scan and clean all infected nodes.
  * Identify and close the initial entry point (patching vulnerabilities or malicious attachments).

* Vulnerability Management:
  * Apply all pending security patches to servers and medical devices.

* Account Sanitization (Legal/Security Audit):
  * Audit Active Directory for any backdoor accounts created during the incident.
  * Revoke unauthorized API keys or third-party integrations.

* Credential Hardening:
  * Implement a mandatory password change for the entire organization.
  * Ensure MFA is enforced on 100% of external-facing applications.

---

### Step 5: Recovery (Recuperacion)

Restore systems to normal operation, ensuring that data is clean, intact, and available for clinical use.

* Phased Restoration:
  * Restore systems by priority: 1. Life-critical systems (ICU), 2. Diagnostic systems (EHR/PACS), 3. Administrative systems.

* Data Integrity Verification (Medical/Legal Accuracy):
  * Perform Sanity Checks on medical records (verify dosages, allergies, and histories).
  * Use Hash functions to prove the restored database has not been altered (Digital Evidence).

* Continuous Monitoring:
  * Implement Enhanced Logging for 30 days to detect lingering malicious activity.

* Service Handover:
  * Coordinate with Hospital Leadership to transition from Paper Protocol back to Digital Systems.

---

### Step 6: Lessons Learned (Post-Incidente y Mejora Continua)

Analyze the incident and fulfill all regulatory reporting obligations.

* Incident Documentation and Legal Reporting:
  * Create a Final Incident Report for insurance and audits.
  * Finalize mandatory notifications to INAI (Mexico) and HHS/OCR (USA) to demonstrate compliance.

* Root Cause Analysis (RCA):
  * Conduct a meeting with IT, Legal, and Medical leadership to identify failures.
  * Evaluate the effectiveness of the Paper Protocol.

* Update Security Controls:
  * Modify firewall rules and SIEM logic based on the incident.
  * Update this Playbook to reflect lessons learned.

* Staff Awareness:
  * Implement targeted training for the department where the breach originated.
