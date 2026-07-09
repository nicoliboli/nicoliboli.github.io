---
layout: post
title: expressway
---

# enumeration

```
$ nmap -p22 -sC -sV 10.129.238.52
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-05 12:32 EST
Nmap scan report for 10.129.238.52
Host is up (0.028s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.71 seconds

$ sudo nmap -sU -T5 10.129.238.52
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-05 12:31 EST
Warning: 10.129.238.52 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.238.52
Host is up (0.032s latency).
Not shown: 855 open|filtered udp ports (no-response), 144 closed udp ports (port-unreach)
PORT    STATE SERVICE
500/udp open  isakmp

Nmap done: 1 IP address (1 host up) scanned in 144.35 seconds

$ sudo nmap -sU --script ike-version -p500 10.129.238.52
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-05 12:36 EST
Nmap scan report for 10.129.238.52
Host is up (0.029s latency).

PORT    STATE SERVICE
500/udp open  isakmp
| ike-version: 
|   attributes: 
|     XAUTH
|_    Dead Peer Detection v1.0

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

$ ike-scan -M -A 10.129.238.52
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Aggressive Mode Handshake returned
        HDR=(CKY-R=5bb81bb137153fa8)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.709 seconds (1.41 hosts/sec).  1 returned handshake; 0 returned notify

$ ike-scan -A --pskcrack  10.129.238.52
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Aggressive Mode Handshake returned HDR=(CKY-R=3ee1ba2ff404acef) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
ee8eeacdabd774a2eff6ec54aca6fb6efcdc13bcb81f552e5c6562debbd7bc114e345d72ad9554813b1e7f869bd15858b42f37c53448c8a12d4b73db8185376b34d69e2456b78a630fc274d3056ad443a6982939fe8074a6a8da31b2b67916377d700ac425b2b9ec8bb2d269e0550d8051d2ff30fa14bbca44ef4c47985fa48c:abec4960c627bcc2657185c0f008ebd9584c0ec5fc8d2092b92ec2fcce2a66e94e18cc138a320171f0d7ffce0684f32e4119a357f7ee7fc8c2235f68771b9346ad509bd20601bd15224be890e703759849b5fe47a4d4fe617d3991bb6a9bd65c9fafe6fb8eb3a0671d16b450952994af4b006700f796f839ae62ad9d33c071ca:3ee1ba2ff404acef:c92246fc6a6b5b7e:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:87fb491c4e7aec283d5ee6d9ef066790127d84f0:4c6d9e81c23275a8c05d387fa70acfe451e15858feb2539f97364f86e0705fa0:f95732250a9a40ba6b3744bd5c2514c99a271831
Ending ike-scan 1.9.6: 1 hosts scanned in 0.051 seconds (19.49 hosts/sec).  1 returned handshake; 0 returned notify
```


https://www.levelblue.com/blogs/spiderlabs-blog/cracking-ike-missionimprobable-part-1