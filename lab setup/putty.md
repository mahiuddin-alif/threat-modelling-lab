# How to Connect PuTTY to Kali Linux

This guide explains how to connect to a Kali Linux machine using PuTTY from Windows.

---

## Requirements

Before starting, make sure you have:

- A Windows computer
- Kali Linux installed (VirtualBox/VMware)
- Network connection between host and Kali Linux
- PuTTY installed

---

# Step 1: Download PuTTY

Go to the official PuTTY website and download the correct version for your system.

🔗 Official Website: https://www.putty.org/

Install PuTTY after downloading.

---

# Step 2: Open PuTTY

After installation:

1. Open **PuTTY**
2. You will see the PuTTY configuration window

Example:

![PuTTY Interface](https://upload.wikimedia.org/wikipedia/commons/0/01/Putty.png)

---

# Step 3: Find Kali Linux IP Address

Open the Kali Linux terminal and run:

```bash
ifconfig
````

or

```bash
ip a
```

Look for the IP address under your network adapter.

Example:

```bash
inet 192.168.1.105
```

---

# Step 4: Check SSH Service

PuTTY uses SSH to connect, so SSH must be enabled in Kali Linux.

Check SSH status:

```bash
service ssh status
```

If SSH is not running, start it:

```bash
service ssh start
```

Enable SSH on boot:

```bash
systemctl enable ssh
```

---

# Step 5: Connect Using PuTTY

1. Open PuTTY
2. Enter the Kali Linux IP address in the **Host Name** field
3. Keep Port as:

```text
22
```

4. Connection type should be:

```text
SSH
```

5. Click **Open**

---

# Step 6: Login to Kali Linux

When prompted:

* Enter your Kali Linux username
* Enter your password

Example:

```text
login as: kali
password: ********
```

After successful login, you will get terminal access to Kali Linux from Windows.

---

# Troubleshooting

## Connection Refused

Make sure SSH service is running:

```bash
service ssh start
```

---

## Host Not Found

* Check the Kali Linux IP address
* Ensure both systems are on the same network

---

## Firewall Issues

Allow SSH through firewall if necessary.

---

# Useful Commands

## Restart SSH

```bash
service ssh restart
```

## Stop SSH

```bash
service ssh stop
```

## Check IP Address

```bash
ip a
```

---

# Conclusion

You have successfully connected PuTTY to Kali Linux using SSH. This setup is useful for remote terminal access, penetration testing labs, and server management.

---

## Tags

`Kali Linux` `PuTTY` `SSH` `VirtualBox` `Cyber Security` `Linux`

```
```

