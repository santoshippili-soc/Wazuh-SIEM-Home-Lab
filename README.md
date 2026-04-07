# 🛡️ Wazuh SIEM Lab — SOC Analyst Home Lab Project

A complete SIEM home lab built with **Wazuh**, demonstrating real-world SOC workflows including endpoint monitoring, File Integrity Monitoring (FIM), threat detection, and alert triage.

---

## 📌 Overview

This project demonstrates a complete SIEM lab using Wazuh with:
- **Ubuntu Server** — Wazuh Manager + Indexer + Dashboard
- **Windows Endpoint** — Wazuh Agent
- **Real-time monitoring** and alert generation

**Wazuh capabilities covered:**
- Log analysis
- File Integrity Monitoring (FIM / Syscheck)
- Intrusion detection
- Vulnerability detection
- Real-time alerting

---

## 🗺️ Architecture

```
Windows Endpoint (Agent)  →  Wazuh Server (Ubuntu)  →  Wazuh Dashboard
         Logs                      Analysis                  Alerts
```

```
┌──────────────────────────────────────────────────────────┐
│                    Home Network (Bridged)                  │
│                                                           │
│  ┌──────────────────────────────────────┐                 │
│  │         Ubuntu Server VM             │                 │
│  │         (VirtualBox - Bridged)        │                 │
│  │                                      │                 │
│  │  ┌────────────────────────────────┐  │                 │
│  │  │     Wazuh Manager (v4.12)      │  │                 │
│  │  │     Wazuh Indexer              │  │                 │
│  │  │     Wazuh Dashboard            │  │                 │
│  │  └──────────────┬─────────────────┘  │                 │
│  └─────────────────┼────────────────────┘                 │
│                    │  Agent communication (port 1514/TCP)  │
│  ┌─────────────────┴────────────────────┐                 │
│  │         Windows Host Machine         │                 │
│  │                                      │                 │
│  │  ┌────────────────────────────────┐  │                 │
│  │  │     Wazuh Agent (Windows)      │  │                 │
│  │  │     FIM: C:\Users\<user>\      │  │                 │
│  │  │          Downloads\WAZUH-TEST  │  │                 │
│  │  └────────────────────────────────┘  │                 │
│  └──────────────────────────────────────┘                 │
└──────────────────────────────────────────────────────────┘
```

| Component | Host | Role |
|---|---|---|
| Wazuh Manager | Ubuntu Server (VirtualBox, Bridged) | Collects, analyzes, and stores data from agents |
| Wazuh Indexer | Ubuntu Server (same VM) | Stores and indexes security events |
| Wazuh Dashboard | Ubuntu Server (same VM) | Web UI for alert visualization and triage |
| Wazuh Agent | Windows (host machine) | Sends logs and system events to the manager |

---

## 💻 Requirements

| Resource | Minimum |
|---|---|
| RAM | 6 GB |
| Storage | 40 GB |
| Virtualization | Oracle VM VirtualBox |
| Ubuntu | 22.04 LTS |
| Windows | Windows 10 / 11 |

---

## ⚙️ Step 1: Install Wazuh Server (Ubuntu)

```bash
sudo apt update && sudo apt upgrade -y

curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

The `-a` flag installs all components: Manager, Indexer, and Dashboard.  
**Save the admin credentials printed at the end of the script — they are only shown once.**

**If the indexer needs manual installation (IPv4 force):**
```bash
sudo apt -o Acquire::ForceIPv4=true install wazuh-indexer=4.12.0-* -y
```

### ✅ Verify Services

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All three should show `active (running)`.

### 🌐 Access Dashboard

```
https://<ubuntu-vm-ip>
```

- Accept the self-signed certificate browser warning
- Log in with the credentials from the install script (username: `admin`)

> ⚠️ **Security note:** Change the default admin password immediately. Never commit credentials to a public repository.

---

## 🖥️ Step 2: Install Windows Agent

Download the MSI installer:
```
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.0-1.msi
```

During installation, set the Manager IP:
```
Manager IP: <WAZUH_SERVER_IP>
```

---

## ⚙️ Step 3: Configure Agent

Edit the agent config file as Administrator:
```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Verify the `<client>` block contains the correct manager address:
```xml
<client>
  <server>
    <address><WAZUH_SERVER_IP></address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

---

## 🔐 Step 4: Register Agent

**On Ubuntu — generate the agent key:**
```bash
sudo /var/ossec/bin/manage_agents
```

- Select `A` → add agent → name it `WINDOWS-AGENT`
- Enter the Windows machine IP
- Select `E` → extract key → copy the full base64 string

**On Windows — apply the key using agent-auth:**
```cmd
"C:\Program Files (x86)\ossec-agent\agent-auth.exe" -m <WAZUH_SERVER_IP>
```

### 🔄 Restart Agent

```cmd
net stop wazuh
net start wazuh
```

### ✅ Verify Connection

On Ubuntu:
```bash
sudo /var/ossec/bin/agent_control -l
```

Expected output:
```
ID: 001, Name: WINDOWS-AGENT, IP: <WINDOWS_IP>, Active
```

---

## 📁 Step 5: Enable File Integrity Monitoring (FIM)

Edit `ossec.conf` on Windows. Inside the `<syscheck>` block, add:

```xml
<directories realtime="yes" recursion_level="5">
  C:\Users\<username>\Downloads\WAZUH-TEST
</directories>
```

Restart the agent to apply:
```cmd
net stop wazuh
net start wazuh
```

---

## 🧪 Step 6: Generate Alerts (SOC Testing)

### 🔹 File Integrity Monitoring

Create, modify, or delete a file inside:
```
C:\Users\<username>\Downloads\WAZUH-TEST
```
➡️ FIM alert appears in the dashboard under **Integrity Monitoring**  
➡️ Rule IDs: `554` (file added), `550` (file modified), `553` (file deleted)

---

### 🔹 Failed Login Detection

Trigger a failed Windows login attempt.  
➡️ Wazuh detects **Event ID 4625** and generates an authentication failure alert

---

### 🔹 PowerShell Activity Monitoring

Run in PowerShell:
```powershell
Get-Process
```
➡️ Windows event logs capture PowerShell execution; Wazuh forwards and alerts

---

### 🔹 Malware Simulation (EICAR Test File)

Download the EICAR anti-malware test file:
```
https://www.eicar.org/download-anti-malware-testfile/
```
➡️ Tests whether Wazuh detects known malware signatures via Syscheck and VirusTotal integration

---

## 🔍 Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Agent shows `Never connected` | Wrong manager IP | Verify `<address>` in `ossec.conf`; restart agent |
| Agent shows `Disconnected` | Manager or agent service down | `sudo systemctl restart wazuh-manager` + `net stop/start wazuh` |
| Dashboard not loading | Service not started | Check `wazuh-dashboard` systemctl status |
| No FIM alerts | Agent not restarted after config change | Restart agent after editing `ossec.conf` |
| Certificate warning | Self-signed cert (expected) | Click Advanced → Proceed in browser |

---

## 🔐 Key Concepts Demonstrated

| Concept | Details |
|---|---|
| SIEM Deployment | Full Wazuh stack via all-in-one installer |
| Agent Enrollment | Key-based agent registration via `manage_agents` |
| File Integrity Monitoring | Real-time Syscheck monitoring with `realtime="yes"` |
| Alert Triage | Viewing and interpreting alerts in the Wazuh Dashboard |
| Threat Simulation | FIM, failed logins, PowerShell activity, EICAR test |
| MITRE ATT&CK Mapping | T1565 (Data Manipulation), T1083 (File Discovery), T1059 (Scripting) |

---

## 📊 Project Outcomes

- ✅ Real-time log monitoring
- ✅ File Integrity Monitoring (FIM)
- ✅ Threat detection alerts
- ✅ Endpoint visibility
- ✅ SOC-ready lab environment

---

## 🚀 Resume Points

- Built a SIEM lab using Wazuh on Ubuntu, integrating Windows endpoint monitoring
- Configured real-time File Integrity Monitoring (FIM) via Wazuh Syscheck
- Simulated real-world attack scenarios (failed logins, PowerShell activity, malware test files)
- Triaged alerts in the Wazuh Dashboard correlating Windows Event IDs to security events

---

## 🧰 Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Wazuh Manager | 4.12 | SIEM core — log analysis, alerting, FIM |
| Wazuh Indexer | 4.12 | Event storage and indexing |
| Wazuh Dashboard | 4.12 | Web-based alert visualization |
| Wazuh Agent | 4.12 | Windows endpoint monitoring |
| Ubuntu Server | 22.04 LTS | Wazuh Manager host (VirtualBox) |
| VirtualBox | 7.x | Virtualization platform |

---

## 📁 Repository Structure

```
wazuh-siem-lab/
│
├── README.md               ← This file
├── installation-guide.md   ← Detailed server installation commands
├── agent-enrollment.md     ← Agent key generation and registration
├── fim-configuration.md    ← FIM/Syscheck setup and verification
├── configs/
│   └── ossec.conf          ← Sample agent configuration file
└── screenshots/            ← Lab evidence screenshots
```

---

## 🔒 Security Best Practices

- Change the default Wazuh admin password immediately after installation
- Never commit passwords, agent keys, or API tokens to a public repository
- Use a `.env` file or local password manager for credential storage
- Keep the Wazuh stack updated to receive the latest detection rules
- Restrict dashboard access to trusted IPs in production environments

---

## 📜 License

This project is for **educational purposes only**. All credentials and IPs shown are placeholders. Never expose real passwords or agent keys in public repositories.
