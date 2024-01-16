---
title: "HackTheBox - Crocodile"
date: 2023-12-27 09:57:00 +0700
categories: [HackTheBox, Starting Point]
tags: [very easy, web, ftp]
image:
    path: /assets/img/hackthebox/starting_points/Crocodile.png
    alt: Crocodile - HackTheBox
---

[**Crocodile**](https://app.hackthebox.com/starting-point) is one of the Starting Points from [**HackTheBox**](https://app.hackthebox.com/), where in CTF Crocodile we will learn about **??**.

## Introduction

- Connect **Crocodile** using **Pwnbox** or **OpenVPN**.
- Spawn machine.

## Enumeration

To check the target connection and port, we can use **Ping** and **Nmap**.

### Ping

After spawn machine, we can start with `ping` Target IP.

```bash
❯ ping 10.129.23.227

PING 10.129.23.227 (10.129.23.227) 56(84) bytes of data.
64 bytes from 10.129.23.227: icmp_seq=1 ttl=63 time=222 ms
64 bytes from 10.129.23.227: icmp_seq=2 ttl=63 time=241 ms
64 bytes from 10.129.23.227: icmp_seq=3 ttl=63 time=571 ms
64 bytes from 10.129.23.227: icmp_seq=4 ttl=63 time=260 ms

--- 10.129.23.227 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 222.356/323.275/570.700/143.454 ms
```

### Nmap

Scan ports using `nmap`, `-sCV` is a combination of `-sC` and `-sV`, where `-sC` displays the script for the port and `-sV` displays the version info for the port, `-T4` to speed up scanning (the higher the faster [0-5]).

```bash
❯ nmap -sCV -T4 10.129.23.227

Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-27 09:35 WIB
Nmap scan report for JSN.JaringanKU (10.129.23.227)
Host is up (0.24s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE    SERVICE      VERSION
21/tcp   open     ftp          vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
80/tcp   open     http         Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
1077/tcp filtered imgames
1688/tcp filtered nsjtp-data
1971/tcp filtered netop-school
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.07 seconds
```

You can see, there are 5 ports and only 2 are open, 21/tcp which is the ftp service and 80/tcp which is the http service.

If analyzed more deeply, in the ftp section `anonymous` login is allowed, and there are 2 files, let's take them.

### FTP

```bash
❯ ftp 10.129.23.227

Connected to 10.129.23.227.
220 (vsFTPd 3.0.3)
Name (10.129.23.227:huda): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.

ftp> get allowed.userlist
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
226 Transfer complete.
33 bytes received in 0,000122 seconds (264 kbytes/s)

ftp> get allowed.userlist.passwd
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
226 Transfer complete.
62 bytes received in 0,000132 seconds (459 kbytes/s)
```

After that read the file.

```
aron
pwnmeow
egotisticalsw
admin
```
{: file='allowed.userlist'}

```
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```
{: file='allowed.userlist.passwd'}

OK, let's try logging in to FTP using one of these users.

```bash
❯ ftp 10.129.23.227

Connected to 10.129.23.227.
220 (vsFTPd 3.0.3)
Name (10.129.23.227:huda): admin
530 This FTP server is anonymous only.
ftp: Login failed.
ftp> 
```

And the result cannot be that only `anonymous` users can log in, it is possible that the user is for the http service. Let's go to the website.

## Foothold

After entering the web, we are presented with the main web display, and after investigating there are no tabs leading to the login page, therefore we will fuzz the web using `gobuster`

![Desktop View](/assets/img/htb/include/Crocodile-1.png){: .w-75}

### Gobuster

`dir` is used to scan directories, `-u` is used to scan the destination URL, `-w` is used as a wordlist/words that may be in the destination URL, and `-x` is used to scan extensions

```bash
❯ gobuster dir -u http://10.129.23.227/ -w /usr/share/dirbuster/directory-list-2.3-small.txt -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.23.227/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/.html                (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 58565]
/login.php            (Status: 200) [Size: 1577]
/assets               (Status: 301) [Size: 315] [--> http://10.129.23.227/assets/]
/css                  (Status: 301) [Size: 312] [--> http://10.129.23.227/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.23.227/js/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
Progress: 7600 / 262995 (2.89%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 7606 / 262995 (2.89%)
===============================================================
Finished
===============================================================
```

We can see, there is login.php. Come on, go back to the website. Login with the username and password provided in the file earlier.

![Desktop View](/assets/img/htb/include/Crocodile-2.png){: .w-75}
![Desktop View](/assets/img/htb/include/Crocodile-3.png){: .w-75}