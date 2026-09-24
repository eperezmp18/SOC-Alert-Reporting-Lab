# Tier-1 SOC Alert Triage & Escalation: Five Ws Reporting

**Analyst:** Elizabeth Perez  
**Date:** September 2026  
**Lab Environment:** [TryHackMe SOC L1: Alert Reporting](https://tryhackme.com/room/socl1alertreporting)  
**Interactive Dashboard:** `https://static-labs.tryhackme.cloud/apps/socl1-alertreporting/`  
**Methodology:** The "Five Ws" Reporting Framework (Who, What, When, Where, Why), NIST SP 800-61  

---

## 1. Project Overview
In a modern Security Operations Center (SOC), an alert triage is incomplete until it has been documented accurately. Once a Tier-1 analyst triages an event, poor documentation forces Tier-2 and Incident Response teams to duplicate efforts, slowing down critical incident containment.

This project demonstrates structured alert documentation, true positive verification, and formal Tier-2 escalation using the TryHackMe SIEM alert management simulation.

---

## 2. Core Framework: The "Five Ws" of Alert Reporting

Every alert report and escalation ticket created in this project adheres to the standard **Five Ws** methodology:

| Element | Focus | Investigation Objective |
| :--- | :--- | :--- |
| **Who** | Identity | Which user, service account, or process initiated the event? |
| **What** | Action | What exact command, payload execution, or anomalous activity took place? |
| **When** | Timeline | Precise UTC timestamps for alert generation, execution, and detection. |
| **Where** | Location | Which internal workstation/server, IP address, external domain, or URL was involved? |
| **Why** | Verdict | Analytical reasoning and evidence justifying a True Positive or False Positive determination. |

---

## 3. Case Investigation 1: Spoofed Phishing Alert Triage

### Scenario
An incoming alert flagged an email that bypassed initial gateway filters: `Email marked as Phishing after Delivery`.

### Alert Metadata & Telemetry
* **Alert Name:** Email marked as Phishing after Delivery
* **Claimed Sender:** `support@microsoft.com`
* **Recipient User:** Internal corporate recipient
* **Subject:** Urgent Notice: Microsoft Account Verification Required
* **Attachment:** Compressed archive (`.zip`) containing an executable payload

<img width="3744" height="568" alt="image" src="https://github.com/user-attachments/assets/0cad16c2-06e6-40df-990e-00e91741c866" />

### Technical Analysis & Artifact Verification
1. **Header Inspection:** Reviewed email authentication headers. Both **SPF (Sender Policy Framework)** and **DKIM (DomainKeys Identified Mail)** failed authentication, proving the sender domain (`microsoft.com`) was spoofed.
2. **Social Engineering Indicators:** Identified urgency triggers ("urgent action required", impending account suspension) designed to coerce rapid credential submission.
3. **Payload Assessment:** The attached archive contained a suspicious executable imitating a Microsoft support diagnostic tool.

### Formal Five Ws Alert Report
* **Who:** External threat actor spoofing `support@microsoft.com` targeting internal employee inbox.
* **What:** Delivery of a malicious phishing email with failed SPF/DKIM verification containing an obfuscated `.zip` attachment.
* **When:** Logged and flagged following user mailbox delivery.
* **Where:** Inbound gateway to internal mailbox; originating from unauthorized external mail-relay IP.
* **Why (Verdict):** **True Positive.** Domain spoofing confirmed via SPF/DKIM failures combined with high-risk file attachment formats and classic social engineering lures.

<img width="3746" height="619" alt="image" src="https://github.com/user-attachments/assets/de52f5bf-80d1-4a32-8c63-c47bfd35a354" />

---

## 4. Case Investigation 2: Web Shell Execution & Tier-2 Escalation

### Scenario
A high-severity alert triggered indicating suspicious command-line activity stemming from a legacy on-premises web/mail server (`Exchange`).

### Technical Analysis & Findings
* **Process Tree Analysis:** The web server worker process spawned command execution shells (`cmd.exe` executing `revshell.exe`).
* **Observed Activity:** Execution of discovery commands, attempted privilege escalation, and network reconnaissance indicating initial stages of lateral movement.
* **Threat Classification:** Active **Web Shell** presence via application exploit.

### Escalation Ticket (Assigned to L2 Analyst)

> **Ticket Title:** [CRITICAL] Confirmed Web Shell Execution on Exchange Server  
> **Status:** In Progress — Escalated to L2 (`E. Fleming`)  
> **Assigned Analyst:** Elizabeth Perez  
>
> **Escalation Summary:**  
> Identified anomalous process execution on the internal Exchange server where the web process initiated `cmd.exe` to spawn `revshell.exe`. Command logs indicate post-exploitation activity including privilege escalation attempts and internal subnet scanning.
>
> **Actions Taken by L1:**  
> 1. Set alert status to **In Progress** and documented initial forensic indicators.  
> 2. Documented confirmed indicators of compromise (IOCs) including process names and parent PIDs.  
> 3. Reassigned ticket directly to on-shift L2 analyst (`E. Fleming`) for deep-dive forensic analysis and host isolation.

---

## 5. SOC Operational Lessons Learned
* **Preserving Context for Tier-2:** Clear Five Ws summaries allow Tier-2 responders to initiate host quarantine and memory forensics immediately without re-verifying basic header checks.
* **Crisis Communication Discipline:** If critical alerts require immediate response and the assigned L2 is unresponsive, Tier-1 analysts must follow an emergency escalation chain (contacting alternate L2s, L3, and the SOC Manager directly) rather than leaving tickets unmonitored.
