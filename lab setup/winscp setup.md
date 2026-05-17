# How to Use WinSCP with Kali Linux for File Sharing

## What is WinSCP?

WinSCP is a free Windows application used for secure file transfer between:

- Windows
- Kali Linux
- Ubuntu/Linux servers
- Virtual machines

It supports:
- SFTP
- SCP
- FTP

WinSCP mainly uses SSH for secure communication.

---

# Requirements

Before starting, you need:

- Windows PC
- Kali Linux installed (VirtualBox or VMware)
- WinSCP installed on Windows
- SSH enabled on Kali Linux

---

# Step 1: Download WinSCP

Official Website:

https://winscp.net/

Install WinSCP normally on Windows.

---

# Step 2: Update Kali Linux

Open Kali terminal and run:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 3: Install OpenSSH Server on Kali

Install SSH server:

```bash
sudo apt install openssh-server -y
```

---

# Step 4: Start SSH Service

Run:

```bash
sudo systemctl start ssh
```

Enable SSH at boot:

```bash
sudo systemctl enable ssh
```

---

# Step 5: Check SSH Status

Run:

```bash
sudo systemctl status ssh
```

If working properly, you will see:

```text
active (running)
```

---

# Step 6: Find Kali Linux IP Address

Run:

```bash
hostname -I
```

Example:

```text
192.168.1.20
```

Save this IP address.

---

# Step 7: Configure VirtualBox Network

If using VirtualBox:

Go to:

```text
VirtualBox → Settings → Network
```

Recommended:
- Adapter 1 → Bridged Adapter

OR

- NAT Adapter

---

# Step 8: Open WinSCP

Launch WinSCP on Windows.

---

# Step 9: Configure WinSCP Connection

Enter the following:

| Field | Value |
|------|------|
| File Protocol | SFTP |
| Host Name | Kali IP Address |
| Port Number | 22 |
| User Name | Kali Username |
| Password | Kali Password |

Example:

| Field | Example |
|------|------|
| Host Name | 192.168.1.20 |
| User Name | kali |
| Password | kali |

---

# Step 10: Login to Kali Linux

Click:

```text
Login
```

First time connection:
- Security warning appears
- Click:
  - Accept

---

# Step 11: WinSCP Interface

WinSCP has two panels:

| Left Panel | Right Panel |
|------------|-------------|
| Windows Files | Kali Linux Files |

---

# Step 12: Upload Files to Kali

To upload:

1. Select file from left side
2. Drag to right side

OR

Click:

```text
Upload
```

---

# Step 13: Download Files from Kali

To download:

1. Select file from right side
2. Drag to left side

OR

Click:

```text
Download
```

---

# Step 14: Create Folders

Right click inside WinSCP:

```text
New → Directory
```

Enter folder name.

---

# Step 15: Edit Files Directly

1. Right click a file
2. Select:
   - Edit

After saving:
- WinSCP automatically updates the file on Kali Linux.

---

# Step 16: Access Root Directory (Optional)

Open terminal:

```bash
sudo passwd root
```

Set root password.

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

# Common Kali Linux Directories

| Directory | Purpose |
|-----------|---------|
| /home/kali | User files |
| /etc | Configuration files |
| /var/log | Log files |
| /usr/share | Installed tools |

---

# Useful Commands

## Restart SSH

```bash
sudo systemctl restart ssh
```

---

## Check IP Address

```bash
ip a
```

---

## Check SSH Port

```bash
sudo ss -tulnp | grep ssh
```

---

# Common Problems

## Connection Refused

Solution:

```bash
sudo systemctl start ssh
```

---

## Authentication Failed

Check:
- Username
- Password
- Keyboard layout

---

## Timeout Error

Check:
- VM network settings
- Firewall
- Correct IP address

---

# Advantages of Using WinSCP

- Secure file transfer
- Drag-and-drop support
- Easy remote file management
- Works with Kali Linux via SSH
- Simple user interface

---

# Conclusion

WinSCP is a simple and secure way to transfer files between Windows and Kali Linux using SFTP and SSH.
