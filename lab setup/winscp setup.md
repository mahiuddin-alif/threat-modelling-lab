# How to Use WinSCP for File Sharing

## What is WinSCP?

WinSCP is a free file transfer software for Windows used to securely transfer files between:

- Windows PC
- Linux Server
- Ubuntu Virtual Machine
- Remote Servers

It supports:
- SCP
- SFTP
- FTP
- WebDAV

WinSCP is commonly used with SSH for secure file sharing.

---

# Requirements

Before using WinSCP, make sure you have:

- Windows computer
- Ubuntu/Linux server or VM
- Internet or local network connection
- SSH enabled on Linux server

---

# Step 1: Download WinSCP

Official Website:

https://winscp.net/

Download and install WinSCP normally.

---

# Step 2: Install OpenSSH Server on Ubuntu

Open Ubuntu terminal and run:

```bash
sudo apt update
sudo apt install openssh-server -y
```

---

# Step 3: Check SSH Status

Run:

```bash
sudo systemctl status ssh
```

If SSH is running, you will see:

```text
active (running)
```

---

# Step 4: Find Ubuntu IP Address

Run:

```bash
hostname -I
```

Example output:

```text
192.168.1.15
```

Save this IP address.

---

# Step 5: Open WinSCP

Launch WinSCP on Windows.

You will see the Login window.

---

# Step 6: Configure Connection

Enter the following information:

| Field | Value |
|------|------|
| File Protocol | SFTP |
| Host Name | Ubuntu IP Address |
| Port Number | 22 |
| User Name | Ubuntu Username |
| Password | Ubuntu Password |

Example:

| Field | Example |
|------|------|
| Host Name | 192.168.1.15 |
| User Name | alif |
| Password | yourpassword |

---

# Step 7: Login to Ubuntu Server

Click:

```text
Login
```

First time connection:
- WinSCP shows SSH security warning
- Click:
  - Accept

---

# Step 8: Understanding WinSCP Interface

WinSCP uses dual panels:

| Left Side | Right Side |
|-----------|------------|
| Windows Files | Ubuntu/Linux Files |

This allows easy drag-and-drop file transfer.

---

# Step 9: Upload Files to Ubuntu

To upload:

1. Select file from left panel
2. Drag file to right panel

OR

Click:

```text
Upload
```

File will be transferred to Ubuntu server.

---

# Step 10: Download Files from Ubuntu

To download:

1. Select file from right panel
2. Drag file to left panel

OR

Click:

```text
Download
```

---

# Step 11: Create Folder

Right click inside WinSCP panel:

```text
New → Directory
```

Enter folder name.

---

# Step 12: Edit Files Directly

1. Right click file
2. Select:
   - Edit

WinSCP opens file in text editor.

After saving:
- WinSCP automatically uploads updated file.

---

# Step 13: Enable Root Access (Optional)

By default, root login may be disabled.

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Find:

```text
PermitRootLogin prohibit-password
```

Change to:

```text
PermitRootLogin yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

# Step 14: Common Linux Directories

| Directory | Purpose |
|-----------|---------|
| /home | User files |
| /var/www/html | Website files |
| /etc | Configuration files |
| /var/log | Log files |

---

# Useful SSH Commands

## Restart SSH

```bash
sudo systemctl restart ssh
```

---

## Check SSH Status

```bash
sudo systemctl status ssh
```

---

## Check Open Ports

```bash
sudo ss -tulnp
```

---

# Common Problems

## Connection Refused

Possible reasons:
- SSH not installed
- SSH service stopped
- Wrong IP address

Solution:

```bash
sudo systemctl start ssh
```

---

## Timeout Error

Check:
- Firewall
- VM Network Adapter
- Correct IP address

---

## Authentication Failed

Check:
- Username
- Password
- Keyboard layout

---

# VirtualBox Network Settings

For Ubuntu VM:

Go to:

```text
VirtualBox → Settings → Network
```

Recommended:
- Adapter 1 → NAT

OR

- Bridged Adapter (for direct local network access)

---

# Advantages of WinSCP

- Easy file transfer
- Secure communication
- Drag-and-drop support
- Supports SSH/SFTP
- Simple interface

---

# Conclusion

WinSCP is one of the easiest and most secure ways to transfer files between Windows and Ubuntu/Linux systems using SSH and SFTP.
