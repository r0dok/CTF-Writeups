 Notes
 - zip2john > file.zip
 - john

- gobuster dir -u TARGET -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 
- find / -type f -regex '.*skat\.txt$' 2> /dev/null                       
- nmap -T4 -A -p- TARGET -oN nmap.txt
- find / -name findme.txt 2> test.txt
- hydra -l mark -P /usr/share/wordlists/rockyou.txt 10.10.116.4 ssh
- Hydra -l admin -x 4:4:MLVICDX -vV sshydra.com  ssh

hydra -l admin -P /usr/share/john/password.lst -vV ftplogin.com ftp

nmap
- O - OS system check
- sV - checking for services
- sS - bypassing the three way handshake
- sn - less in depth

Random points from THM
 - Popup's (<script>alert(“Hello World”)</script>) - Creates a Hello World message popup on a users browser.
 - Writing HTML (document.write) - Override the website's HTML to add your own (essentially defacing the entire page).
 - XSS Keylogger (http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js.html) - You can log all keystrokes of a user, capturing their password and other sensitive information they type into the webpage.
 - Port scanning (http://www.xss-payloads.com/payloads/scripts/portscanapi.js.html) - A mini local port scanner (more information on this is covered in the TryHackMe XSS room).
