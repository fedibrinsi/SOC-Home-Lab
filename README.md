
# 📘 SOC Home Lab

*A complete, real-world SOC environment combining Wazuh, TheHive, Cortex, MISP, AD DS, and Tailscale.*


## 🚀 Overview

This project is a full **Security Operations Center (SOC) Home Lab**.
It simulates a real enterprise SOC by integrating:

* **Wazuh SIEM** (log collection, IDS, FIM, vulnerability detection)
* **TheHive** (incident response platform)
* **Cortex** (analysis & automation engine)
* **MISP** (threat intelligence platform)
* **Active Directory** (Windows enterprise environment simulation)
* **Tailscale** (private mesh VPN)

The goal is to build a fully functional detection & response pipeline:

```
Wazuh → TheHive → Cortex → MISP → Operator
```

This repository documents the entire setup, architecture, steps, configurations, and integration testing.

---

# 🏗️ Architecture Diagram

```
                +---------------------+
                |   Active Directory  |
                | Windows Server 2019 |
                +----------+----------+
                           |
                           | Windows Events (via Wazuh Agent)
                           v
                +---------------------+
                |       Wazuh        |
                | OVA Deployment     |
                | Indexer / Manager  |
                +----------+----------+
                           |
                           | Alerts (custom integration)
                           v
                +---------------------+
                |      TheHive        |
                |  Case Management    |
                +----------+----------+
                           |
                           | API Call (Analyzers, Responders)
                           v
                +---------------------+
                |       Cortex        |
                |  Threat Analysis    |
                +----------+----------+
                           |
                           | TAXII / API
                           v
                +---------------------+
                |        MISP         |
                | Threat Intelligence |
                +---------------------+
```

---

# 🛠️ Technologies Used

| Component            | Role                                                     |
| -------------------- | -------------------------------------------------------- |
| **Wazuh**            | SIEM, IDS, FIM, Vulnerability detection                  |
| **Active Directory** | Authentication, domain management, enterprise simulation |
| **TheHive**          | Incident response & case management                      |
| **Cortex**           | Analyzer & responder automation                          |
| **MISP**             | Threat intelligence sharing & IOCs                       |
| **GNS3**             | Network simulation                                       |
| **Tailscale**        | Mesh VPN for isolated communication                      |
| **Docker**           | Deployment of TheHive, Cortex, MISP                      |
| **Wazuh OVA**        | Stable SIEM environment                                  |

---

# 🔧 Environments Setup

## 1. 🗂️ Active Directory Lab

* Installed **Windows Server 2019** (domain controller)
* Installed **Windows 10** client VM
* Configured:

  * Static IP
  * DNS pointing to domain controller
  * Forest: `latifa.local`
  * Organizational Units (OU): `FST`
  * Users: `Firas FB Brinsi`, `Sonia`, etc.
  * Groups: `Students`
* Enabled Windows Security Auditing
* Joined client machines to AD domain

🔍 **Logs collected by Wazuh confirmed successful AD integration.**

---

## 2. 📊 Wazuh SIEM Deployment

### **Attempt 1 — Docker (Failed)**

* Physical machine lacked CPU/RAM
* Indexer & Manager containers kept stopping
* SSL errors (`curl: (35) SSL_ERROR_SYSCALL`)

### **Attempt 2 — VM (Partially Failed)**

* Dashboard unreachable due to wrong port mappings / NAT mode

### **Final Deployment — ✔️ Official OVA**

* Imported Wazuh OVA in VirtualBox
* Started services manually:

```bash
sudo systemctl start wazuh-indexer wazuh-manager wazuh-dashboard
sudo systemctl enable wazuh-indexer wazuh-manager wazuh-dashboard
```

* Stable environment
* Dashboard accessible

---

## 3. 🎯 Installing Wazuh Agent on Active Directory

* Installed agent on Windows Server
* Linked to Wazuh Manager
* Created user in AD → appeared in Wazuh logs
* Validated:

  * Event ID 4720 (user creation)
  * Event ID 4728/4732 (group assignment)

✔️ AD → Wazuh communication = Working.

---

## 4. 🔐 Tailscale Private Network

* Installed Tailscale on the Windows Server
* Joined a shared network with teammates
* Validated connectivity via `ping`
* Used to simulate secure remote SOC access

---

## 5. 🐝 Deployment of TheHive (Incident Response Platform)

### **Technologies**

* Docker
* Cassandra
* Elasticsearch (adjusted permissions via `chown -R`)

### **Tested features**

* User creation
* Organization creation
* Analyzer execution
* API workflow
* Case creation

---

## 6. 🤖 Cortex Deployment (Automation Engine)

* Installed via Docker
* Configured API keys
* Connected Cortex to TheHive
* Tested:

  * Analyzers
  * Responders
  * Logs + error handling

---

## 7. 🛰️ MISP Deployment (Threat Intelligence Platform)

* Deployed using Docker (testing folder, not production)
* Created users & organizations
* Added:

  * Events
  * Attributes (IPs, hashes, domains)
  * Taxonomies

---

## 8. 🔗 TheHive ↔ Cortex ↔ MISP Integration

* Added Cortex URL + API key inside TheHive
* Linked MISP to Cortex analyzers
* Tested automated IOC enrichment
* Simulated a full SOC workflow:

  1. Create case in TheHive
  2. Add observable
  3. Launch Cortex Analyzer
  4. Fetch intelligence from MISP
  5. Enrich case

✔️ Integration validated end-to-end.

---

## 9. 🔥 Wazuh → TheHive Integration (Custom)

### **Step 1 — Install Python Library**

```
cd /var/ossec/integrations
sudo /var/ossec/framework/python/bin/pip3 install thehive4py==1.8.1
```

### **Step 2 — Add Scripts**

Two files inside `/var/ossec/integrations/`:

* `custom-w2thive.py`
* `custom-w2thive`

### **Step 3 — Set Permissions**

```bash
sudo chmod 755 custom-w2thive.py custom-w2thive
sudo chown root:ossec custom-w2thive.py custom-w2thive
```

### **Step 4 — Update Wazuh Config**

```xml
<integration>
  <name>custom-w2thive</name>
  <hook_url>http://TheHive_Server_IP:9000</hook_url>
  <api_key>API_KEY_OF_FEDI</api_key>
  <alert_format>json</alert_format>
</integration>
```

### **Result**

Any alert generated by Wazuh is automatically pushed to TheHive as a **case** or **alert**.

✔️ Communication validated using `curl -I http://thehive_ip:9000`.

---


# 🔮 Future Improvements

* Add Suricata IDS to simulate network attacks
* Add Zeek for behavioral network analytics
* Automate deployments using Ansible
* Add dashboards with Grafana
* Simulate phishing, brute-force, malware events

