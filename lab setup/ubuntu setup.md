Here's a **modified version** of your Ubuntu installation lab with improved structure, additional details, and better formatting for GitHub. Changes include: clearer section numbering, added verification steps, more troubleshooting options, and a **Lab Completion Checklist** at the end.

---

```markdown
# Week 1 Lab: Installing Ubuntu on VirtualBox for Wazuh

## Learning Outcomes

By the end of this lab, you will be able to:

- Create and configure a virtual machine in VirtualBox
- Install Ubuntu Server/Desktop manually without unattended automation
- Configure network settings (Bridged vs. NAT) for host-to-VM access
- Enable and verify OpenSSH server for remote access
- Retrieve and document the VM's IP address for future connections
- Perform basic post-installation verification

## Objective

Set up a fully functional Ubuntu virtual machine in VirtualBox that will serve as the host for the Wazuh SIEM platform. This VM must be accessible from Windows via PuTTY and WinSCP for subsequent security monitoring tasks.

## Scenario

You are a security analyst tasked with deploying an open-source SIEM solution (Wazuh) in a lab environment. Before installing Wazuh, you need a stable Linux base. Your organization uses Windows workstations, so you must configure the Ubuntu VM with proper networking to allow remote management from Windows using SSH and SCP.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **VirtualBox** | Installed on Windows/macOS/Linux – [Download](https://www.virtualbox.org/) |
| **Ubuntu ISO** | 22.04 LTS (Server or Desktop) – [Download](https://ubuntu.com/download/server) |
| **RAM** | At least 8 GB available on host machine |
| **Disk Space** | At least 25 GB free |
| **Virtualization** | Enabled in BIOS (Intel VT-x / AMD-V) |

---

## Lab Duration

**Estimated time:** 20–30 minutes

---

## Step 1: Create a New Virtual Machine

1. Launch **Oracle VM VirtualBox**.
2. Click the **New** button (or press `Ctrl + N`).
3. In the "Create Virtual Machine" dialog, enter:

   | Field | Value |
   |-------|-------|
   | **Name** | `Ubuntu-Wazuh` |
   | **Folder** | Your preferred location (e.g., `C:\VMs\`) |
   | **ISO Image** | Click dropdown → **Choose a disk file** → Select your Ubuntu ISO |
   | **Skip Unattended Installation** | ✅ **UNCHECK** this box |

4. Click **Next**.

---

## Step 2: Allocate Hardware Resources

Configure the following settings:

| Setting | Recommended Value | Minimum |
|---------|-------------------|---------|
| **Memory (RAM)** | 8192 MB (8 GB) | 4096 MB (4 GB) |
| **CPU Cores** | 2 | 1 |
| **Disk Size** | 25 GB | 20 GB |

**Additional disk options:**
- **Hard disk type:** `VDI` (VirtualBox Disk Image) or `VMDK`
- **Storage on physical hard disk:** `Dynamically allocated`

Click **Next** → **Finish**.

---

## Step 3: Configure Network for Windows Access

> ⚠️ **Critical step** – This determines whether Windows can reach your Ubuntu VM.

1. Select your `Ubuntu-Wazuh` VM.
2. Click **Settings** → **Network**.
3. **Adapter 1** tab:
   - ✅ **Enable Network Adapter** (checked)
   - **Attached to:** Choose one:

| Mode | When to Use | Windows Can Reach Ubuntu? |
|------|-------------|---------------------------|
| **Bridged** | Recommended for PuTTY/WinSCP/Wazuh | ✅ Yes (by IP) |
| **NAT** | Ubuntu needs internet only | ❌ No (requires port forwarding) |

4. Click **OK**.

---

## Step 4: Install Ubuntu

### 4.1 Start the VM

1. Click **Start** to boot the VM.
2. Wait for the Ubuntu installer to load.

### 4.2 Installation Walkthrough

Follow these prompts carefully:

| Step | Selection |
|------|-----------|
| **Language** | `English` |
| **Keyboard layout** | As preferred (e.g., English US) |
| **Type of installation** | `Minimal` (or `Ubuntu Server` if using server ISO) |
| **Network setup** | Accept DHCP defaults (automatic IP) |
| **Storage configuration** | `Use Entire Disk` (default) |
| **Storage encryption** | `Skip` (not needed for lab) |

### 4.3 Profile Setup

Create your user account:

| Field | Example Value |
|-------|---------------|
| **Your name** | `Wazuh User` |
| **Computer name** | `wazuh-server` |
| **Username** | `wazuh-user` |
| **Password** | Choose a strong password (save it!) |

### 4.4 SSH Setup (Very Important!)

- ✅ **CHECK** `Install OpenSSH server`
- This allows PuTTY and WinSCP connections later

### 4.5 Featured Server Snaps

- Select **None** (skip this section)

### 4.6 Begin Installation

1. Click **Install**.
2. Wait 5–10 minutes for completion.

---

## Step 5: First Boot & Post-Installation

1. When installation finishes, click **Reboot Now**.
2. The VM will restart.
   - If prompted to "Remove installation medium," press `Enter`.
3. Log in with your username (`wazuh-user`) and password.

---

## Step 6: Verify Ubuntu is Ready

### 6.1 Get the IP Address

Inside the Ubuntu VM, run:

```bash
ip a
```

Look for `inet` under `eth0` or `enp0s3`:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic enp0s3
```

📌 **Save this IP address** – Example: `192.168.1.100`

### 6.2 Verify SSH is Running

```bash
sudo systemctl status ssh
```

Look for `active (running)`.

### 6.3 Test Network Connectivity

From your **Windows** machine, open Command Prompt (`cmd`) and run:

```cmd
ping 192.168.1.100
```

Expected output:

```
Reply from 192.168.1.100: bytes=32 time<1ms TTL=64
```

---

## Verification Checklist

Before proceeding to Week 2, confirm:

- [ ] Ubuntu VM boots successfully
- [ ] You can log in with `wazuh-user` credentials
- [ ] IP address is visible via `ip a`
- [ ] SSH service shows `active (running)`
- [ ] Windows can ping the Ubuntu IP address
- [ ] You have saved the Ubuntu IP address somewhere accessible

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Can't see IP address in `ip a`** | Run `sudo dhclient` to request IP from DHCP |
| **Ubuntu installer doesn't start** | Enable virtualization in BIOS (Intel VT-x / AMD-V) |
| **Windows can't ping Ubuntu IP** | Change VirtualBox network from NAT to **Bridged** |
| **SSH connection refused** | Run `sudo systemctl enable ssh --now` and `sudo ufw allow 22` |
| **VM is very slow** | Allocate more RAM (8 GB) or CPU cores (2+) |
| **"No bootable medium found"** | Re-attach the ISO in VM Settings → Storage |
| **Ubuntu asks for login loop** | You set a username different from `wazuh-user` – use that instead |

---

## Quick Reference Commands

```bash
# Check IP address
ip a | grep inet

# Check SSH status
sudo systemctl status ssh

# Restart SSH if needed
sudo systemctl restart ssh

# Update system (optional after install)
sudo apt update && sudo apt upgrade -y

# Check if firewall is blocking
sudo ufw status
```

---

## Lab Completion Checklist

| Task | Completed |
|------|-----------|
| Virtual machine created in VirtualBox | ☐ |
| Ubuntu installed with OpenSSH server | ☐ |
| Network set to Bridged mode | ☐ |
| IP address documented: `_______________` | ☐ |
| Windows can ping Ubuntu VM | ☐ |
| SSH service verified as running | ☐ |

---

## Next Steps

✅ **Ubuntu is now installed and ready for remote access.**

Proceed to:

- **Week 2 Lab:** Configuring PuTTY and WinSCP on Windows
- **Week 3 Lab:** Installing Wazuh SIEM on Ubuntu

---

## Additional Resources

- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [OpenSSH Documentation](https://www.openssh.com/manual.html)
```

---

## Summary of Changes Made

| Original | Modified Version |
|----------|------------------|
| Plain list of prerequisites | Table format with download links |
| No time estimate | Added "Lab Duration: 20–30 minutes" |
| Basic numbered steps | Sub-steps (4.1, 4.2, etc.) for better readability |
| No verification commands | Added ping test from Windows |
| No checklist | Added "Verification Checklist" and "Lab Completion Checklist" |
| Brief troubleshooting | Expanded troubleshooting table |
| Basic next steps | Added "Additional Resources" section |

Let me know if you want me to create modified versions for **PuTTY/WinSCP** and **Wazuh installation** as well.
