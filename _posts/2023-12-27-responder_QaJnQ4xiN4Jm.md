---
title: "HackTheBox - Responder (In Progress)"
date: 2023-12-27 19:00:00 +0700
categories: [HackTheBox, Starting Point]
tags: [very easy, web, smb, winrm]
image:
    path: /assets/img/htb/Responder.png
    alt: Responder - HackTheBox
---

[**Responder**](https://app.hackthebox.com/starting-point) is one of the Starting Points from [**HackTheBox**](https://app.hackthebox.com/), where in CTF Responder we will learn about **??**.

## Introduction

- Connect **Responder** using **Pwnbox** or **OpenVPN**.
- Spawn machine.

## Enumeration

To check the target connection and port, we can use **Ping** and **Nmap**.

### Ping

After spawn machine, we can start with `ping` Target IP.

```bash
❯ ping 10.129.110.220

PING 10.129.110.220 (10.129.110.220) 56(84) bytes of data.
64 bytes from 10.129.110.220: icmp_seq=1 ttl=127 time=289 ms
64 bytes from 10.129.110.220: icmp_seq=2 ttl=127 time=412 ms
64 bytes from 10.129.110.220: icmp_seq=3 ttl=127 time=424 ms
64 bytes from 10.129.110.220: icmp_seq=4 ttl=127 time=255 ms

--- 10.129.110.220 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 255.356/344.882/423.771/73.870 ms
```

### Nmap

Scan ports using `nmap`, `-p-` to scan all ports, `-sV` to display port version info, `-T4` to speed up scanning (the higher the faster [0-5]).

```bash

```