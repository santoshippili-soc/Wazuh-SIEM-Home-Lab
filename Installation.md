# Installation Guide — Wazuh SIEM Stack on Ubuntu

Full step-by-step commands for deploying the Wazuh Manager, Indexer, and Dashboard on an Ubuntu Server VM.

---

## Environment

| Setting | Value |
|---|---|
| Host OS | Windows (VirtualBox host) |
| VM OS | Ubuntu Server 20.04+ |
| VirtualBox Network | Bridged Adapter |
| Wazuh Version | 4.12 |

---

## Step 1 — Prepare Ubuntu VM

Ensure the VM has internet access:
```bash
ping -c 3 google.com
```

Update packages:
```bash
sudo apt update && sudo apt upgrade -y
```

Check the VM's IP address (note this — needed for dashboard access and agent config):
```bash
ifconfig
# or
ip a
```

---

## Step 2 — Add Wazuh GPG Key

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o \
  /usr/share/keyrings/wazuh-archive-keyring.gpg
```

This downloads and stores the GPG key used to verify Wazuh package authenticity.

---

## Step 3 — Run the All-in-One Installation Script

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
```

**Flags:**
| Flag | Meaning |
|---|---|
| `-a` | Install all components: Manager, Indexer, Dashboard |
| `-i` | Interactive mode — pauses for confirmation at key steps |

**What gets installed:**
- `wazuh-manager` — core SIEM engine
- `wazuh-indexer` — OpenSearch-based event storage
- `wazuh-dashboard` — web UI (Kibana-based)

**At the end of the script, credentials are displayed:**
```
User: admin
Password: <generated-password>
```
Save these immediately. They are only shown once.

---

## Step 4 — Manual Indexer Install (if needed)

If the indexer step fails due to IPv6 issues, force IPv4:
```bash
sudo apt -o Acquire::ForceIPv4=true install wazuh-indexer=4.12.0-* -y
```

---

## Step 5 — Verify Services Are Running

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All three should show `active (running)`.

---

## Step 6 — Access the Dashboard

Open a browser and navigate to:
```
https://<ubuntu-vm-ip>
```

- Accept the self-signed certificate warning
- Log in with `admin` and the password from the install script output

> ⚠️ **Never share or commit the generated password.** Treat it like a production secret.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Dashboard not loading | Check `wazuh-dashboard` service status; ensure port 443 is not blocked |
| Indexer connection errors | Check `wazuh-indexer` service; try the IPv4 force install command |
| Certificate warning in browser | Expected for self-signed cert — click Advanced → Proceed |
| Agent shows Disconnected | Verify Ubuntu VM IP hasn't changed; check firewall allows port 1514 |
