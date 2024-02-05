---
title: "HackTheBox - RegistryTwo"
date: 2024-02-05 08:35:00 +0700
categories: [HackTheBox, Machines]
tags: [insane]
---

![Desktop View](/assets/img/hackthebox/machines/insane/registrytwo/RegistryTwo.png)

## Enumeration
### Ping
```bash
PING 10.10.11.223 (10.10.11.223) 56(84) bytes of data.
64 bytes from 10.10.11.223: icmp_seq=1 ttl=63 time=98.0 ms
64 bytes from 10.10.11.223: icmp_seq=2 ttl=63 time=121 ms
64 bytes from 10.10.11.223: icmp_seq=3 ttl=63 time=43.8 ms
64 bytes from 10.10.11.223: icmp_seq=4 ttl=63 time=63.6 ms

--- 10.10.11.223 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 43.782/81.580/120.937/29.871 ms
```
### Nmap
```bash
# Nmap 7.94 scan initiated Mon Feb  5 08:23:50 2024 as: nmap -sCV -T4 -oN nmap.txt 10.10.11.223
Nmap scan report for 10.10.11.223
Host is up (0.061s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
22/tcp   open  ssh                OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fa:b0:03:98:7e:60:c2:f3:11:82:27:a1:35:77:9f:d3 (RSA)
|   256 f2:59:06:dc:33:b0:9f:a3:5e:b7:63:ff:61:35:9d:c5 (ECDSA)
|_  256 e3:ac:ab:ea:2b:d6:8e:f4:1f:b0:7b:05:0a:69:a5:37 (ED25519)
443/tcp  open  ssl/http           nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to https://www.webhosting.htb/
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=free-hosting/stateOrProvinceName=Berlin/countryName=DE
| Not valid before: 2023-02-01T20:19:22
|_Not valid after:  2024-02-01T20:19:22
5000/tcp open  ssl/http           Docker Registry (API: 2.0)
|_http-title: Site doesn''t have a title.
| ssl-cert: Subject: commonName=*.webhosting.htb/organizationName=Acme, Inc./stateOrProvinceName=GD/countryName=CN
| Subject Alternative Name: DNS:webhosting.htb, DNS:webhosting.htb
| Not valid before: 2023-03-26T21:32:06
|_Not valid after:  2024-03-25T21:32:06
5001/tcp open  ssl/commplex-link?
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=*.webhosting.htb/organizationName=Acme, Inc./stateOrProvinceName=GD/countryName=CN
| Subject Alternative Name: DNS:webhosting.htb, DNS:webhosting.htb
| Not valid before: 2023-03-26T21:32:06
|_Not valid after:  2024-03-25T21:32:06
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Mon, 05 Feb 2024 01:24:52 GMT
|     Content-Length: 10
|     found
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Date: Mon, 05 Feb 2024 01:24:21 GMT
|     Content-Length: 26
|     <h1>Acme auth server</h1>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Date: Mon, 05 Feb 2024 01:24:22 GMT
|     Content-Length: 26
|_    <h1>Acme auth server</h1>
| tls-alpn: 
|   h2
|_  http/1.1
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5001-TCP:V=7.94%T=SSL%I=7%D=2/5%Time=65C038C5%P=x86_64-pc-linux-gnu
SF:%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:
SF:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20
SF:Bad\x20Request")%r(GetRequest,8E,"HTTP/1\.0\x20200\x20OK\r\nContent-Typ
SF:e:\x20text/html;\x20charset=utf-8\r\nDate:\x20Mon,\x2005\x20Feb\x202024
SF:\x2001:24:21\x20GMT\r\nContent-Length:\x2026\r\n\r\n<h1>Acme\x20auth\x2
SF:0server</h1>\n")%r(HTTPOptions,8E,"HTTP/1\.0\x20200\x20OK\r\nContent-Ty
SF:pe:\x20text/html;\x20charset=utf-8\r\nDate:\x20Mon,\x2005\x20Feb\x20202
SF:4\x2001:24:22\x20GMT\r\nContent-Length:\x2026\r\n\r\n<h1>Acme\x20auth\x
SF:20server</h1>\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clos
SF:e\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:
SF:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\
SF:x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf
SF:-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalSer
SF:verCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20te
SF:xt/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x2
SF:0Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCo
SF:ntent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n
SF:\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x20R
SF:equest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\
SF:x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,A7,"HTTP/1\
SF:.0\x20404\x20Not\x20Found\r\nContent-Type:\x20text/plain;\x20charset=ut
SF:f-8\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Mon,\x2005\x20Feb
SF:\x202024\x2001:24:52\x20GMT\r\nContent-Length:\x2010\r\n\r\nNot\x20foun
SF:d\n")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Typ
SF:e:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x
SF:20Bad\x20Request")%r(LDAPSearchReq,67,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20c
SF:lose\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb  5 08:25:58 2024 -- 1 IP address (1 host up) scanned in 128.79 seconds
```