---
layout: post
title: facts
---


# enumeration

```
$ nmap -T5 -p- 10.129.17.204
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-05 08:01 EST
Nmap scan report for facts.htb (10.129.17.204)
Host is up (0.030s latency).
Not shown: 65111 closed tcp ports (conn-refused), 421 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
54321/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 17.30 seconds

Nmap done: 1 IP address (1 host up) scanned in 0.59 seconds

$ nmap -sC -sV -p22,80 10.129.17.204
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-05 07:58 EST
Nmap scan report for 10.129.17.204
Host is up (0.028s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp open  http    nginx 1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
|_http-server-header: nginx/1.26.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.72 seconds
```

since i can see there is a redirect, i need to update my `/etc/hosts` file to associate the domain `facts.htb` with the ip `10.129.17.204` 

```
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.129.17.204   facts.htb facts
```

## port 80 - http

open `burpsuite` and navigate to `facts.htb` with firefox and get an error that the browser isn't supported. try with chrome and see that it works. 

![facts_ff_visit](/assets/img/facts_ff_visit.png)

![facts_chrome_visit](/assets/img/facts_chrome_visit.png)

my guess is that it's filtering on the user-agent string, so i need to keep this in mind if i need to script anything. my user-agent string is `Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36` and i go that by hitting `F12` to get to the developer tools (or inspect) window, clicking on the `Network` tab, and looking at the request headers

![facts_user_agent](/assets/img/facts_user_agent.png)

now i need to setup my proxy in chrome, so that i can see the traffic in `burpsuite` (there might be an extension for this, but this is quick b/c i'm on a linux system). close chrome and run it via the terminal with the argument `--proxy-server=http://127.0.0.1:8080`. the sytax is `protocol://ip:port`

```
$ google-chrome --proxy-server=http://127.0.0.1:8080
...
```

clicked aroung the site a bit and ran `sqlmap` using the search query as a parameter, but didn't find much. run `gobuster` and find an `admin page`

![facts_admin](/assets/img/facts_admin.png)

try logging in with `admin:admin` creds, but get an error.

![facts_admin_login](/assets/img/facts_admin_login.png)

click the `Forgot your password?` link try to send a password reset link to `admin@facts.htb`, but get an error of `Not found email address`

![facts_password_reset_admin](/assets/img/facts_password_reset_admin.png)

i remember from clicking around that there were names associated with the comments made on the different posts. at this point i could script it using the sitemap, but it's not a ton of pages, so i just went through and made a manual list. added a couple default creds too and appended `@facts.htb` because i saw the `contact@facts.htb` email 

```
bob
carol
dave
jean
root
admin
```

none of those work, so try to create me own account with creds `test:test`. i can log in, but can't do anything. on the `profile` page i do see a `#ID` of 5, which makes me think there are 4 other users

![facts_profile](/assets/img/facts_profile.png)

search for any exploits related to the application running (`Camaleon CMS version 2.9.0`) and found one

![facts_privesc_cms](/assets/img/facts_privesc_cms.png)

update my passord to `test1`

![facts_password_update](/assets/img/facts_password_update.png)