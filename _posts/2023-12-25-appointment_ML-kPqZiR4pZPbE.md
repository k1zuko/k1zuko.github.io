---
title: "HackTheBox - Appointment"
date: 2023-12-25 14:17:00 +0700
categories: [HackTheBox, Starting Point]
tags: [very easy, web, databases, injection, sql, sql injection]
image:
    path: /assets/img/htb/Appointment.png
    alt: Appointment - HackTheBox
---

[**Appointment**](https://app.hackthebox.com/starting-point) is one of the Starting Points from [**HackTheBox**](https://app.hackthebox.com/), where in CTF Fawn we will learn about **SQL (Sctuctured Query Language), SQL Injection**.

## Introduction

- Connect **Appointment** using **Pwnbox** or **OpenVPN**.
- Spawn machine.

## Enumeration

To check the target connection and port, we can use **Ping** and **Nmap**.

### Ping

After spawn machine, we can start with `ping` Target IP.

```bash
❯ ping 10.129.173.189

PING 10.129.173.189 (10.129.173.189) 56(84) bytes of data.
64 bytes from 10.129.173.189: icmp_seq=1 ttl=63 time=283 ms
64 bytes from 10.129.173.189: icmp_seq=2 ttl=63 time=305 ms
64 bytes from 10.129.173.189: icmp_seq=3 ttl=63 time=327 ms
64 bytes from 10.129.173.189: icmp_seq=4 ttl=63 time=247 ms

--- 10.129.173.189 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 247.321/290.647/327.390/29.608 ms
```

### Nmap 

Scan ports using `nmap`, `-sCV` is a combination of `-sC` and `-sV`, where `-sC` displays the script for the port and `-sV` displays the version info for the port, `-T4` to speed up scanning (the higher the faster [0-5]).

```bash
❯ nmap -sCV -T4 10.129.173.189

Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-26 17:44 WIB
Nmap scan report for JSN.JaringanKU (10.129.173.189)
Host is up (0.22s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.15 seconds
```

You can see the open port 80/tcp, which is the http service port. You can open it in your browser.

## Foothold

After entering the website, we are presented with a login page. In this position, we don't know what to log in with. Therefore, this time we will use SQL injection to be able to log into the website.

![Desktop View](/assets/img/htb/include/Appointment-1.png){: .w-75}

### SQL Injection

default credentials:

```
admin::admin
guest::guest
user::user
root::root
administrator::password
```

However, these credentials cannot be used, therefore we will use SQL injection.

In SQL injection there are many ways that can be used. But this time, we use simple SQL injection with `admin'#` as the username and the password is up to you. Where the function of `'` is to end strings and `#` is used to comment the next command, so that way all that is read is the user as `admin` and the password will be ignored because it has already been commented.

![Desktop View](/assets/img/htb/include/Appointment-2.png){: .w-75}