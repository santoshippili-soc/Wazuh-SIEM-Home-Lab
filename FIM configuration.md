# File Integrity Monitoring (FIM) — Configuration & Verification

Configuring Wazuh's Syscheck module to monitor a Windows directory in real-time and verifying alerts in the dashboard.

---

## What is FIM (Syscheck)?

Wazuh's **Syscheck** module monitors files and directories for changes including:
- File **creation**
- File **modification** (content, permissions, ownership)
- File **deletion**

For each event, Wazuh records: file path, timestamp, MD5/SHA1/SHA256 hash, inode, permissions, and the type of change.

**MITRE ATT&CK relevance:**
| Technique | ID | How FIM helps |
|---|---|---|
| Data Manipulation | T1565 | Detects unauthorized file modifications |
| File and Directory Discovery | T1083 | Flags unusual file access patterns |
| Indicator Removal | T1070 | Catches log/evidence deletion |

---

## Configuration — Windows Agent

### 1. Open the Agent Config File

Open Notepad **as Administrator** and edit:
```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

### 2. Add the Monitored Directory

Locate the `<syscheck>` block. Add your monitored directory inside it:

```xml
<syscheck>
  <!-- Existing config above -->

  <!-- Add this line: -->
  <directories realtime="yes">C:\Users\<your-username>\Test</directories>

</syscheck>
```

Replace `<your-username>` with your actual Windows username.

**Key attribute — `realtime="yes"`:**
- Without this, Syscheck only scans on the default schedule (every 12 hours)
- With `realtime="yes"`, changes trigger an alert **immediately**

### Additional Syscheck Options

```xml
<!-- Monitor with file checksum reporting -->
<directories realtime="yes" report_changes="yes" check_all="yes">
  C:\Users\<your-username>\Test
</directories>
```

| Attribute | Effect |
|---|---|
| `realtime="yes"` | Immediate alerts on file changes |
| `report_changes="yes"` | Includes a diff of what changed inside text files |
| `check_all="yes"` | Checks hash, size, permissions, owner, and modification time |

---

### 3. Restart the Wazuh Agent

**Via Services (Windows):**
1. Open `services.msc`
2. Find **Wazuh** service
3. Right-click → **Restart**

**Via Wazuh Agent Manager GUI:**
1. Open Wazuh Agent Manager from Start Menu
2. Click **Restart**

**Via PowerShell (Admin):**
```powershell
Restart-Service -Name WazuhSvc
```

---

## Verification — Triggering FIM Alerts

### Test 1 — Create a File
```cmd
echo "Test content" > C:\Users\<your-username>\Test\testfile.txt
```

### Test 2 — Modify a File
```cmd
echo "Modified content" >> C:\Users\<your-username>\Test\testfile.txt
```

### Test 3 — Delete a File
```cmd
del C:\Users\<your-username>\Test\testfile.txt
```

---

## Viewing Alerts in the Dashboard

1. Open the Wazuh Dashboard: `https://<ubuntu-vm-ip>`
2. Navigate to **Agents** → select your Windows agent
3. Go to **Integrity Monitoring**
4. Each file operation should appear as an alert entry with:
   - **Rule ID:** 550 (file modified), 554 (file added), 553 (file deleted)
   - **File path:** full path of the changed file
   - **MD5/SHA1/SHA256:** hashes before and after (for modifications)
   - **Timestamp:** exact time of the event

### Example Alert Fields

| Field | Example Value |
|---|---|
| Rule ID | 554 |
| Rule description | File added to the system |
| File | `C:\Users\santosh\Test\testfile.txt` |
| Agent | WINDOWS-AGENT |
| SHA256 (after) | `a3f5c...` |
| Timestamp | 2026-04-01T18:30:00Z |

---

## Troubleshooting FIM

| Issue | Likely Cause | Fix |
|---|---|---|
| No alerts appearing | Agent not restarted after config change | Restart Wazuh agent service |
| Alerts delayed | `realtime="yes"` missing | Add the attribute and restart |
| Path not found error | Typo in directory path | Verify path exists; use backslashes `\` |
| Agent Disconnected | Manager IP changed | Update manager IP in agent config |

---

## Manager-Side FIM Log

On the Ubuntu Manager, FIM events are logged to:
```bash
sudo tail -f /var/ossec/logs/alerts/alerts.log | grep syscheck
```

This shows raw alert data as it arrives from the agent in real-time.
