# Agent Enrollment Guide — Windows Agent → Wazuh Manager

Steps to register a Windows machine as a monitored Wazuh agent.

---

## Overview

Agent enrollment involves two sides:
1. **Manager side (Ubuntu)** — generate a unique authentication key for the agent
2. **Agent side (Windows)** — install the agent software, apply the key, connect to the manager

---

## Step 1 — Install Wazuh Agent on Windows

1. Download the latest Wazuh Agent MSI from the official docs:  
   https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

2. Run the MSI installer with default settings

3. After install, the **Wazuh Agent Manager** GUI is accessible from the Start Menu

---

## Step 2 — Generate Agent Key on Ubuntu Manager

Run the agent management utility:
```bash
sudo /var/ossec/bin/manage_agents
```

Follow the interactive prompts:

```
****************************************
* Wazuh v4.x.x Agent manager          *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist all agents (L).
   (R)emove an agent (R).
   (Q)uit.

Choose your action: A a
```

- Enter a name for the agent (e.g., `WINDOWS-AGENT`)
- Leave IP address as `any` unless you want to lock to a specific IP
- Confirm the agent is created

Then extract the key:
```
Choose your action: E e
Available agents:
   ID: 001, Name: WINDOWS-AGENT, IP: any
Provide the ID of the agent to extract the key (or [Enter] to cancel): 001
```

Copy the full base64 key string that is displayed.

> ⚠️ **Never commit this key to a public repository.** It authenticates the agent to your manager. If exposed, regenerate it immediately using option `R` (remove) then `A` (add again).

---

## Step 3 — Apply Key on Windows Agent

1. Open **Wazuh Agent Manager** from the Start Menu
2. Click **Authentication key** tab
3. Paste the copied key into the key field
4. Click **Save**
5. In the **Configuration** tab, enter the Ubuntu Manager IP address
6. Click **Save** → **Restart**

---

## Step 4 — Verify Agent Status

On the Wazuh Dashboard:
- Navigate to **Agents**
- Your `WINDOWS-AGENT` should appear with status **Active**
- If it shows **Never connected**, check that port 1514 (UDP/TCP) is open between Windows host and Ubuntu VM

---

## Useful Manager-Side Commands

```bash
# List all registered agents
sudo /var/ossec/bin/agent_control -l

# Check agent connection status
sudo /var/ossec/bin/agent_control -s

# Restart the Wazuh manager
sudo systemctl restart wazuh-manager

# View manager logs
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Security Note

Agent keys are sensitive credentials. Best practices:
- Regenerate keys if they are accidentally exposed
- Use IP-locked agent registration in production (`<agent-ip>` instead of `any`)
- Rotate keys periodically in high-security environments
