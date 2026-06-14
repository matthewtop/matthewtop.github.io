---
title: Husar-security-auditor
date: 2026-04-10
description: '"A lightweight Linux security auditing tool written in Python, focused on practical host hardening, network exposure analysis, and privilege-aware configuration checks.'
---
### Overview 
**HUSAR** is a lightweight Linux security auditing tool focused on practical host hardening checks. It scans local system configuration, services, networking, users, and filesystem posture, then prints a clear terminal report with severity levels and a security score. 
### Key Capabilities 
* **SSH Hardening:** Scans root login status, password authentication, empty-password policies, and dynamic context-aware behavior (skips deep SSH config findings if the service is inactive). 
* **Users & Authentication:** Checks for extra UID 0 accounts, duplicate UIDs, and tracks password policy hints from `/etc/login.defs`. 
* **Network Exposure:** Monitors open/listening ports and flags high-risk exposed services (FTP, Telnet, DB ports, RPC, SMB) along with basic SYN cookies posture. 
* **Firewall & Service Hygiene:** Automatically detects active backends (`firewalld`, `UFW`, `nftables`, `iptables`) and contextually filters out noisy false positives. 
* **Filesystem & Kernel Hardening:** Evaluates risky permissions on critical files (`/etc/shadow`, `/etc/passwd`), analyzes SUID footprint, detects basic encryption signals (`dm-crypt`/`LUKS`), and audits selected `sysctl` controls.