# Wazuh Setup Guide (Ubuntu 22.04 LTS Server)

## What is Wazuh?

Wazuh is an open-source security platform used for:

- SIEM (Security Information and Event Management)
- Threat Detection
- Log Monitoring
- Intrusion Detection
- Vulnerability Detection
- File Integrity Monitoring

Wazuh works with:
- Wazuh Server
- Wazuh Dashboard
- Wazuh Agents

---

# System Requirements

## Minimum Requirements

- Ubuntu 22.04 LTS Server
- 4 GB RAM
- 2 CPU Cores
- 50 GB Storage
- Internet Connection

Recommended:
- 8 GB RAM or more

---

# Step 1: Update Ubuntu Server

Run:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Required Packages

```bash
sudo apt install curl apt-transport-https unzip wget lsb-release gnupg -y
```

---

# Step 3: Download Wazuh Installation Script

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

Give execute permission:

```bash
chmod +x wazuh-install.sh
```

---

# Step 4: Install Wazuh All-in-One

This installs:
- Wazuh Indexer
- Wazuh Manager
- Wazuh Dashboard

Run:

```bash
sudo ./wazuh-install.sh -a
```

Installation may take:
- 10 to 30 minutes

---

# Step 5: Save Generated Credentials

After installation, Wazuh shows dashboard credentials.

Example:

```text
Username: admin
Password: xxxxxxxx
```

Save the password carefully.

---

# Step 6: Access Wazuh Dashboard

Open browser:

```text
https://YOUR_SERVER_IP
```

Example:

```text
https://192.168.1.10
```

Because of self-signed SSL certificate:
- Click Advanced
- Continue to website

Login using:
- Username: admin
- Password: generated password

---

# Step 7: Check Wazuh Service Status

## Check Wazuh Manager

```bash
sudo systemctl status wazuh-manager
```

---

## Check Wazuh Dashboard

```bash
sudo systemctl status wazuh-dashboard
```

---

## Check Wazuh Indexer

```bash
sudo systemctl status wazuh-indexer
```

---

# Step 8: Enable Services on Boot

```bash
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-dashboard
sudo systemctl enable wazuh-indexer
```

---

# Step 9: Install Wazuh Agent (Optional)

Install agents on other machines to monitor them.

---

## Linux Agent Installation

### Add Repository Key

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
```

---

### Add Repository

```bash
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

---

### Install Agent

Replace:
- WAZUH_SERVER_IP with your server IP

```bash
sudo WAZUH_MANAGER='WAZUH_SERVER_IP' apt-get install wazuh-agent -y
```

Example:

```bash
sudo WAZUH_MANAGER='192.168.1.10' apt-get install wazuh-agent -y
```

---

### Start Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

# Step 10: Open Firewall Ports

If UFW firewall is enabled:

```bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 443/tcp
```

Reload firewall:

```bash
sudo ufw reload
```

---

# Important Wazuh Ports

| Port | Purpose |
|------|----------|
| 443 | Dashboard |
| 1514 | Agent Communication |
| 1515 | Agent Registration |
| 9200 | Indexer API |

---

# Useful Commands

## Restart Wazuh Services

```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-dashboard
sudo systemctl restart wazuh-indexer
```

---

## Check Logs

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Check Agent Status

```bash
sudo /var/ossec/bin/agent_control -l
```

---

# Common Problems

## Dashboard Not Opening
Check:
- Server IP
- Firewall
- Wazuh dashboard service

---

## Agent Not Connecting
Check:
- Correct server IP
- Port 1514 and 1515 open
- Agent service status

---

## Low Performance
Increase:
- RAM
- CPU cores
- Storage

---

# Architecture Overview

```text
+----------------+
| Wazuh Dashboard|
+----------------+
         |
         v
+----------------+
| Wazuh Manager  |
+----------------+
         |
         v
+----------------+
| Wazuh Indexer  |
+----------------+
         |
         v
+----------------+
| Wazuh Agents   |
+----------------+
```

---

# Features of Wazuh

- Real-time monitoring
- Malware detection
- Log analysis
- Rootkit detection
- Compliance monitoring
- File integrity monitoring
- Active response system

---

# Conclusion

Wazuh is a powerful open-source cybersecurity platform widely used for SIEM, monitoring, and threat detection. It is suitable for cybersecurity labs, enterprise monitoring, and learning SOC operations.
