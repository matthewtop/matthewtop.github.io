---
tags:
  - htb
  - linux
  - ssh
  - port forwarding
draft: false
date: 2026-07-06
title: dynamic-port-forwarding-with-ssh-and-socks-tunneling
---

### Q1: You have successfully captured credentials to an external facing Web Server. Connect to the target and list the network interfaces. How many network interfaces does the target web server have? (Including the loopback interface)

We can start by connecting to the host using SSH:

```
ssh ubuntu@$ipAddress$
```

after successful login we can check the list of network interfaces and get the answer:

```
ubuntu@WEB01:~$ ifconfig
```

### Q2: Apply the concepts taught in this section to pivot to the internal network and use RDP (credentials: victor:pass@123) to take control of the Windows target on $ipAddress$. Submit the contents of Flag.txt located on the Desktop.

We can begin with enabling dynamic port forwarding with SSH:

```
ssh -D 9050 ubuntu@$ipAddress$
```

after successful login we can set `proxychains.conf` to use `socks4`:

```
sudo nano /etc/proxychains.conf
```

We need to make sure that it is set properly and has this line:
```
socks4 127.0.0.1 9050
```

After setting everything we can try to run `nmap` scan on already logged host if it is enabling `RDP`:

```
proxychains nmap -v -Pn -sT $ipAddress$
```

We can use `RDP` so let's connect and get the flag:
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:. /cert:ignore
```

