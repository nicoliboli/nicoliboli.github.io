---
layout: post
title: preignition
categories: htb
tags:
- htb
- starting_point
- preignition
author: nicoliboli
description: starting point | tier 0 | box 5
date: '2025-06-16 13:53:14 -0400'
---
# enumeration

```
$ nmap 10.129.204.26                                                                                          
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:42 EDT
Nmap scan report for 10.129.204.26
Host is up (0.095s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 9.91 seconds

$ nmap -sC -sV -p 80 10.129.204.26
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-16 13:42 EDT
Nmap scan report for 10.129.204.26
Host is up (0.096s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.14.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.98 seconds
```

## port 80 - http

visit the page in the browser and see the welcome page.

![nginx_server](/assets/img/preignition_nginx_server.png)

try `gobuster` to enumerate other directories that might be lying around

```
$ gobuster dir -u http://10.129.204.26 -w /usr/share/wordlists/dirb/common.txt   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.204.26
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin.php            (Status: 200) [Size: 999]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
```

visit the page and see that it lives up to its name as a login portal.

![preignition_admin_page](/assets/img/preignition_admin_page.png)

# exploitation

try the once common default creds `admin:admin` and get a successful login with the flag

![preignition_admin_login](/assets/img/preignition_admin_login.png)