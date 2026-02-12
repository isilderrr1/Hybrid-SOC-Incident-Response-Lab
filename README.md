# Hybrid-SOC-Incident-Response-Lab
A hybrid security lab combining physical network infrastructure with a centralized SOC and Incident Response system.
# 🛡️ Hybrid Network SOC & Incident Response Lab

This repository documents the design and implementation of a **hybrid security lab** that integrates physical network infrastructure with a centralized **SOC (Security Operations Center)** and **Incident Response** system.

## 🏗️ 1. Physical Architecture (Hardware Layer)

The lab is designed to simulate a real enterprise environment, monitoring the traffic passing through network nodes.

### 🔌 Physical Infrastructure & Connections
The setup bridges a standard home gateway with a dedicated laboratory subnet:

**Internet** <---> **TIM Hub** (Gateway) <---> **TP-Link Managed Switch**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+---> **DGA4132** (LAB Router: 192.168.50.1)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+---> **Vodafone Station** (AP LAB)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+---> **MacBook Pro** (SOC Server & Wi-Fi Client)

### Components Detail:
* **TIM Hub**: Main gateway providing external connectivity.
* **TP-Link Managed Switch**: The backbone of the lab, distributing connections to all network nodes.
* **DGA4132**: Lab Router managing the **192.168.50.0/24** subnet (DHCP & Routing).
* **Vodafone Station**: Configured as a **LAN-to-LAN Access Point**, providing Wi-Fi connectivity for lab devices.
* **MacBook Pro (Server/Endpoint)**: Connected via Ethernet for SIEM management (Docker) and monitoring.

## 🛠️ 2. Technology Stack (Software Layer)

-   **SIEM Manager**: [Wazuh (v4.7.2)](https://wazuh.com/) running in a Docker container.
-   **HIDS Custom**: **Argus-Eye**, a specialized monitoring tool for process and service integrity.
-   **Agent**: **Wazuh Agent** installed on the MacBook host for real-time telemetry.

## 🚀 3. Implementation and Monitoring Logic

### A. Data Collection (Log Ingestion)
To integrate the custom tool with the SIEM, a **cronjob** was implemented to bridge Argus-Eye logs into the system's `syslog`:

```bash
# Log injection every minute
* * * * * /home/antonio/.local/bin/argus logs --last 1 | logger
B. Analysis and Decoding
A custom XML decoder was developed on the Wazuh Manager to parse Argus-Eye strings:

Pattern: Detects SEC- (Security) and HEA- (Health) headers.

Extraction: Extracts specific event codes like SEC-04 (Unauthorized Service Detected).

C. Detection Rules & Incident Response
A Level 12 (Critical) rule (ID: 100100) was configured to trigger immediate alerts upon unauthorized process detection.

📊 4. Results & Gallery
The lab successfully detects threats in real-time, providing deep visibility through the Wazuh Dashboard.

SOC Dashboard Overview
The dashboard clearly displays the critical alert (Level 12), highlighting the immediate need for analyst intervention.

Event Detailed Analysis
Wazuh's decoder correctly identifies the SEC-04 code from the raw logs injected via syslog.

Physical Lab Setup
(Includi qui la tua foto del modem/switch quando la caricherai)

🔧 Troubleshooting Note
During implementation, several challenges were addressed:

Regex Optimization: Resolved syntax conflicts in analysisd for accurate log parsing.

Permissions: Configured sudoers and Docker socket permissions for seamless log flow.

Driver Persistence: Fixed Wi-Fi stability issues on MacBook Pro 8,1 (Ubuntu) by forcing b43 module persistence.

🔮 Future Roadmap
[ ] Implement Active Response to automatically kill processes flagged by Argus-Eye.

[ ] Integrate Kali Linux for penetration testing simulation.

[ ] Deploy Metasploitable2 as a vulnerable target within the lab subnet.
