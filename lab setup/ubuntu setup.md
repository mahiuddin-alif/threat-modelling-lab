# Ubuntu Setup Using VirtualBox (Simple & Basic)

## What is VirtualBox?
VirtualBox is a free virtualization software that allows you to run another operating system inside your current OS.

Example:
- Run Ubuntu inside Windows
- Run Linux without deleting Windows

---

# Requirements

- 8 GB RAM recommended
- 25 GB free storage
- Stable internet connection
- VirtualBox software
- Ubuntu ISO file

---

# Step 1: Download VirtualBox

Download from the official website:

https://www.virtualbox.org/

Install it like normal software.

---

# Step 2: Download Ubuntu ISO

Download Ubuntu Desktop ISO:

https://ubuntu.com/download/desktop

Recommended:
- Ubuntu LTS version

---

# Step 3: Create a New Virtual Machine

1. Open VirtualBox
2. Click **New**
3. Enter:
   - Name: Ubuntu
   - Type: Linux
   - Version: Ubuntu (64-bit)

Click **Next**

---

# Step 4: Allocate RAM

Recommended:
- Minimum: 4096 MB (4 GB)
- Better Performance: 8192 MB (8 GB)

Click **Next**

---

# Step 5: Create Virtual Hard Disk

1. Select:
   - **Create a virtual hard disk now**
2. Click **Create**

### Choose:
- VDI (VirtualBox Disk Image)

Click **Next**

### Storage Type:
- Dynamically allocated

Click **Next**

### Disk Size:
- Recommended: 25 GB or more

Click **Create**

---

# Step 6: Attach Ubuntu ISO

1. Select your Ubuntu VM
2. Click **Settings**
3. Go to **Storage**
4. Under Controller IDE/SATA:
   - Click Empty
   - Click CD icon
   - Choose Ubuntu ISO file

Click **OK**

---

# Step 7: Start Ubuntu VM

1. Select Ubuntu VM
2. Click **Start**

Ubuntu installer will boot.

---

# Step 8: Install Ubuntu

1. Select **Install Ubuntu**
2. Choose language
3. Select keyboard layout
4. Connect internet (optional)

---

# Step 9: Installation Type

Choose:

- **Erase disk and install Ubuntu**

Do not worry:
- It only erases the virtual disk
- Your real Windows files remain safe

Click **Install Now**

---

# Step 10: Create User Account

Enter:
- Your Name
- Username
- Password

Click **Continue**

---

# Step 11: Finish Installation

1. Wait for installation
2. Click **Restart Now**
3. Press Enter if needed

Ubuntu setup is complete.

---

# Step 12: Install VirtualBox Guest Additions

Guest Additions improve:
- Screen resolution
- Fullscreen support
- Mouse integration
- Clipboard sharing

## Install Steps

From VirtualBox menu:

```text
Devices → Insert Guest Additions CD Image
```

Then run in Ubuntu terminal:

```bash
sudo apt update
sudo apt install build-essential dkms linux-headers-$(uname -r)
```

Mount and install Guest Additions:

```bash
sudo mkdir /media/cdrom
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/VBoxLinuxAdditions.run
```

Restart Ubuntu:

```bash
sudo reboot
```

---

# Useful Ubuntu Commands

```bash
pwd       # Show current directory
ls        # List files
cd        # Change directory
mkdir     # Create folder
clear     # Clear terminal
```

---

# Update Ubuntu

```bash
sudo apt update
sudo apt upgrade
```

---

# Install Basic Software

```bash
sudo apt install git
sudo apt install vlc
```

---

# Common Problems

## Black Screen
- Increase Video Memory from VM Settings
- Disable 3D Acceleration

## Slow Performance
- Increase RAM
- Increase CPU cores from Settings → System → Processor

## No Fullscreen
- Install Guest Additions

---

# Conclusion

Using Ubuntu in VirtualBox is one of the safest and easiest ways to learn Linux without changing your main operating system.
