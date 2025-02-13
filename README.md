# Centralized Security Monitoring System

## Overview
This project integrates multiple security monitoring tools—**Wazuh, Checkmk, MikroTik, and Elastic Stack**—to provide a **complete security monitoring solution** for system and network defense. It includes **log collection, anomaly detection, system monitoring, and real-time alerting** for enhanced security visibility.

## Components

### 1️⃣ Wazuh – Security Monitoring & Threat Detection
- Centralized **log monitoring** for Windows and Linux systems.
- **File Integrity Monitoring (FIM)** for critical system directories.
- **Custom security rules** for detecting authentication failures, admin privilege escalation, and system modifications.
- **Slack Integration** for real-time security alerts.

### 2️⃣ Elastic Stack – Log Aggregation & Analysis
- **Elasticsearch** for storing and indexing security logs.
- **Kibana** for visualizing security events and trends.
- **Filebeat & Logstash** for collecting and processing logs from various sources.

### 3️⃣ Checkmk – System & Service Monitoring
- **Real-time monitoring** of system health and services.
- **Integration with Wazuh & Elastic Stack** for unified threat intelligence.

### 4️⃣ MikroTik – Network Flow & Syslog Analysis
- **NetFlow monitoring** for analyzing network traffic patterns.
- **Syslog forwarding** to detect suspicious activity.

---

## 📂 Project Structure
```
/Centralized-Security-Monitoring-System
│── Wazuh/                # Wazuh setup, custom rules, and Slack integration
│   ├── EECE_655L_Project.docx # EECE 655L-specific configurations and enhancements
│── Elastic-stack/        # Elasticsearch, Kibana, Filebeat, and Logstash setup
│── Checkmk/              # Checkmk server and agent configuration
│── Mikrotik/             # MikroTik NetFlow and Syslog integration
│── README.md             # Main documentation
```

---

## 📖 Documentation
Each section contains a **detailed README** explaining the installation and configuration of its respective component.

- **[Wazuh Setup](Wazuh/README.md)**
- **[Elastic Stack Setup](Elastic-Stack/README.md)**
- **[Checkmk Setup](Checkmk/README.md)**
- **[MikroTik Setup](Mikrotik/README.md)**

---

## 🔧 General Overview of Setup Instructions

### 1️⃣ Deploy Wazuh Manager
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -o
```

### 2️⃣ Install Wazuh Agents on Target Systems
```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.0-1_amd64.deb && sudo WAZUH_MANAGER='192.168.153.135' WAZUH_AGENT_NAME='ELK-LINUX' dpkg -i ./wazuh-agent_4.9.0-1_amd64.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### 3️⃣ Install Elastic Stack (Elasticsearch + Kibana)
```bash
sudo apt-get update && sudo apt-get install elasticsearch kibana
```

### 4️⃣ Install Checkmk
```bash
sudo apt install ./check-mk-raw-2.3.0p18_0.jammy_amd64.deb
sudo omd create mkmonitoring
sudo omd start
```

### 5️⃣ Configure MikroTik NetFlow & Syslog
- Set up NetFlow in MikroTik’s **Traffic Flow** settings.
- Forward logs via **Syslog** to the Logstash server.

---

## 🎯 Purpose
This project is a **homelab setup** for exploring **security monitoring and threat detection** using **Wazuh, Elastic Stack, Checkmk, and MikroTik**. It focuses on **log analysis, system monitoring, and alerting**, simulating real-world security operations in a controlled environment. The goal is to gain hands-on experience with **blue team techniques**, improve **visibility into network and system events**, and refine **incident detection and response skills**.

---
