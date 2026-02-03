# TryHackMe — RootMe Walkthrough

## Enumeration

Started with Nmap:
```bash
nmap -sV TARGET_IP
```

![Nmap scan](pictures/Untitled%202.png)

Found 2 open ports:
- Port 22: SSH (OpenSSH 7.6p1)
- Port 80: HTTP (Apache 2.4.41)

Visited the web server:

![Website](pictures/Untitled%204.png)

Checked page source for clues:

![Source code](pictures/Untitled%205.png)

Nothing useful. Ran Gobuster:
```bash
gobuster dir -u http://TARGET_IP -w directory-list-lowercase-2.3-small.txt
```

![Gobuster results](pictures/Untitled%206.png)

Found `/uploads` and `/panel`.

**Task 2 Answers:**
- Open ports: `2`
- Apache version: `2.4.41`
- Service on port 22: `ssh`
- Hidden directory: `/panel/`

## Getting a Shell

Checked `/panel/` - file upload form:

![Upload form](pictures/Untitled%209.png)

Checked `/uploads/` - directory listing:

![Uploads directory](pictures/Untitled%2010.png)

Used Burpsuite to analyze traffic. Found PHP session cookie confirming PHP backend:

![Burpsuite](pictures/Untitled%2011.png)

Generated reverse shell from revshells.com:

![Revshells](pictures/Untitled%2012.png)

Created the shell file:

![Creating shell](pictures/Untitled%2013.png)

![Shell code](pictures/Untitled%2014.png)

Attempted upload with `.php` extension - blocked:

![PHP blocked](pictures/Untitled%2015.png)

Renamed to `.php5` to bypass filter - success:

![Upload success](pictures/Untitled%2016.png)

Confirmed upload in `/uploads/`:

![Shell uploaded](pictures/Untitled%2017.png)

Set up listener and triggered shell:

![Shell caught](pictures/Untitled%2018.png)

Confirmed access as www-data:

![whoami](pictures/Untitled%2019.png)

Searched for user flag:
```bash
find / -type f -name user.txt 2>/dev/null
```

![Find user.txt](pictures/Untitled%2020.png)

![User flag](pictures/Untitled%2021.png)

**Task 3 Answer:**
- user.txt: `THM{y0u_g0t_a_sh3ll}`

## Privilege Escalation

Searched for SUID binaries:
```bash
find / -perm -u=s -type f 2>/dev/null
```

![SUID search](pictures/Untitled%2022.png)

Found `/usr/bin/python` with SUID - this should never have SUID in production.

![Python SUID confirmed](pictures/Untitled%2025.png)

Checked GTFObins for python SUID exploit:

![GTFObins](pictures/Untitled%2027.png)

Executed the exploit:
```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![Root shell](pictures/Untitled%2028.png)

**Task 4 Answers:**
- Weird SUID file: `/usr/bin/python`
- root.txt: `THM{pr1v1l3g3_3sc4l4t10n}`
