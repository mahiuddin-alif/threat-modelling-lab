# Ubuntu 22.04 LTS Server Setup Using VirtualBox

## What is Ubuntu Server?
Ubuntu Server is a lightweight Linux operating system mainly used for:

- Web servers
- Databases
- Cloud computing
- Networking
- Development environments

Unlike Ubuntu Desktop, it does not have a graphical interface by default.

---

# Requirements

- VirtualBox installed
- Ubuntu 22.04 LTS Server ISO
- Minimum 4 GB RAM
- 20 GB free storage

---

# Step 1: Download VirtualBox

Official Website:

https://www.virtualbox.org/

Install VirtualBox normally.

---

# Step 2: Download Ubuntu 22.04 LTS Server ISO

Official Download Link:

https://ubuntu.com/download/server

Download:
- Ubuntu Server 22.04 LTS

---

# Step 3: Create New Virtual Machine

1. Open VirtualBox
2. Click **New**

### Enter:
- Name: Ubuntu Server 22.04
- Type: Linux
- Version: Ubuntu (64-bit)

Click **Next**

---

# Step 4: Allocate RAM

Recommended:
- Minimum: 2048 MB
- Better: 4096 MB

Click **Next**

---

# Step 5: Create Virtual Hard Disk

1. Select:
   - **Create a virtual hard disk now**
2. Click **Create**

### Choose:
- VDI (VirtualBox Disk Image)

Click **Next**

### Storage:
- Dynamically allocated

Click **Next**

### Disk Size:
- 20 GB or more

Click **Create**

---

# Step 6: Attach Ubuntu Server ISO

1. Select VM
2. Click **Settings**
3. Go to **Storage**
4. Select **Empty**
5. Click CD icon
6. Choose Ubuntu Server ISO file

Click **OK**

---

# Step 7: Start the VM

1. Select VM
2. Click **Start**

Ubuntu Server installer will boot.

---

# Step 8: Select Language

Choose your preferred language.

Example:
- English

Press **Enter**

---

# Step 9: Installer Options

### Keyboard Layout
Choose keyboard layout.

Example:
- English (US)

---

# Step 10: Network Configuration

Ubuntu usually detects network automatically.

Select:
- Done

---

# Step 11: Configure Proxy

If you do not use proxy:
- Leave empty
- Select **Done**

---

# Step 12: Configure Ubuntu Archive Mirror

Default mirror is fine.

Select:
- Done

---

# Step 13: Storage Configuration

Select:

- **Use an entire disk**

Do not worry:
- Only the virtual disk is affected

Select:
- Done → Continue

---

# Step 14: Create User

Enter:
- Your Name
- Server Name
- Username
- Password

Example:

```text
Name: Alif
Server Name: ubuntu-server
Username: alif
Password: ********
```

Select:
- Done

---

# Step 15: Install OpenSSH Server

IMPORTANT:

Check:
- Install OpenSSH Server

This allows remote SSH access.

Select:
- Done

---

# Step 16: Featured Server Snaps

Optional:
- Docker
- Kubernetes
- PostgreSQL

You can skip for now.

Select:
- Done

---

# Step 17: Finish Installation

Wait for installation.

Then select:
- Reboot Now

Press Enter if installer asks to remove installation media.

---

# Step 18: Login to Ubuntu Server

Enter:
- Username
- Password

You will see terminal like:

```bash
alif@ubuntu-server:~$
```

Ubuntu Server setup is complete.

---

# Basic Linux Commands

```bash
pwd       # Show current directory
ls        # List files
cd        # Change directory
mkdir     # Create folder
rm        # Remove file
clear     # Clear terminal
```

---

# Update Ubuntu Server

```bash
sudo apt update
sudo apt upgrade
```

---

# Install Useful Packages

```bash
sudo apt install net-tools
sudo apt install git
sudo apt install curl
```

---

# Check IP Address

```bash
ip a
```

or

```bash
hostname -I
```

---

# Shutdown or Restart

```bash
sudo shutdown now
sudo reboot
```

---

# SSH Access Example

From another Linux system:

```bash
ssh username@ip-address
```

Example:

```bash
ssh alif@192.168.1.5
```

---

# Common Problems

## Internet Not Working
- Go to:
  - VM Settings → Network
- Enable:
  - NAT Adapter

---

## Slow Performance
- Increase:
  - RAM
  - CPU cores

---

## Login Failed
- Check username/password carefully
- Linux passwords are case-sensitive

---

# Conclusion

Ubuntu 22.04 LTS Server is stable, lightweight, and widely used for servers, networking, cybersecurity, and development environments.
