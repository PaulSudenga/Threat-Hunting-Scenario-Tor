# 🕵️‍♂️ Threat Hunting Scenario: Tor Browser Usage

![image](https://github.com/user-attachments/assets/b3f11519-dcd1-48c4-af69-46abd5431c42)


🚧 *This project is currently in development. Updates coming soon.*

## Overview

This project simulates a real-world threat hunting scenario focused on identifying unauthorized usage of the **Tor Browser** within a corporate environment. While Tor provides anonymity, its use on enterprise systems can be a red flag indicating attempts to bypass network monitoring or conduct illicit activities.

The scenario is built using a cloud-hosted Windows 10 virtual machine and leverages Microsoft security tools to detect, investigate, and respond to potentially malicious behavior.

---

## 🔧 Tools & Technologies

- **Windows 10 (Azure VM)** – Simulated endpoint for suspicious activity
- **Tor Browser** – Used to replicate anonymized browsing behavior
- **Microsoft Defender for Endpoint (MDE)** – Endpoint detection and response
- **Microsoft Sentinel** – SIEM used for log collection, correlation, and investigation
- **KQL (Kusto Query Language)** – Used to query and analyze log data

---

## 🎯 Project Goals

- Simulate the download and use of the Tor Browser on an enterprise Windows 10 system
- Detect key artifacts and behaviors using Microsoft Defender for Endpoint
- Query logs and identify anomalies using KQL in Microsoft Sentinel
- Walk through the full threat hunting process, including:
  - Alert triage
  - Behavioral analysis
  - Threat investigation
  - Response recommendations

---

## 📌 Status

- ✅ Repository initialized
- 🖥️ Azure VM deployment completed
- 🔍 Log simulation and KQL queries coming soon

---

> 🛡️ This project is part of my Home Enterprise Cybersecurity Lab — designed to mirror real-world detection and response workflows using industry-standard Microsoft tools.
