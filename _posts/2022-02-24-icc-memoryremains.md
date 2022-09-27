---
layout: post
title: ICC - Memory Remains
date: 2022-02-24 17:10 +0200
author: Arcee
tags: challenge memory 
categories: [ICC]
---
Challenge of the training platform for ICC.

## Description: 
You have been given a memory dump of the computer that was acquired from the hacker.
Also an encrypted file "secret" was found on the USB stick that was attached to the PC

Question
Memory dump and secret file has been uploaded to:

http://workshop-static.cybexer.io/files.zip

Download the archive, unzip and analyze.
Post the URL as an answer!

### Files
We get two files: memory.dump and secret. As this is a memory dump file we can use Volatility to see if we can get some more information from this. 

`./volatility_2.6_lin64_standalone imageinfo -f memory.dump`

```
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/danique/Documents/Tools/volatility_2.6_lin64_standalone/memory.dump)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002c110f0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002c12d00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-11-06 09:01:00 UTC+0000
     Image local date and time : 2020-11-06 11:01:00 +0200
```
It states a lot of Win7XXX as a suggested profiles, so this is probably a memory dump of a system running Microsoft Windows 7. 

We now check what processes are in the process list: (using pslist which prints all the running processes.)
`./volatility_2.6_lin64_standalone -f memory.dump --profile=Win7SP1x64 pslist`

Which gives: 
```
Volatility Foundation Volatility Framework 2.6
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa800cd20870 System                    4      0     84      507 ------      0 2020-11-06 08:57:55 UTC+0000                                 
0xfffffa800dc0d930 smss.exe                256      4      2       29 ------      0 2020-11-06 08:57:55 UTC+0000                                 
0xfffffa8016fff060 csrss.exe               340    332      9      330      0      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800cd5b060 wininit.exe             388    332      3       74      0      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa80152b7060 csrss.exe               400    380      8      436      1      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800ea0eb10 winlogon.exe            440    380      3      113      1      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800ea3d720 services.exe            484    388      6      186      0      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800ea40b10 lsass.exe               492    388      6      571      0      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800e102790 lsm.exe                 500    388      9      194      0      0 2020-11-06 08:57:56 UTC+0000                                 
0xfffffa800eadbb10 svchost.exe             604    484      9      352      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800eb10550 svchost.exe             672    484      6      256      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800eb3eb10 svchost.exe             724    484     18      449      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800eb6d600 svchost.exe             812    484     17      428      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800eb72b10 svchost.exe             900    484     12      279      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800ebc73f0 svchost.exe             932    484     27      929      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800ebd6b10 svchost.exe             988    484      5      106      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800ebedb10 svchost.exe             548    484     20      572      0      0 2020-11-06 08:57:57 UTC+0000                                 
0xfffffa800ec6a900 spoolsv.exe            1044    484     12      269      0      0 2020-11-06 08:57:58 UTC+0000                                 
0xfffffa800ec7db10 svchost.exe            1092    484     17      300      0      0 2020-11-06 08:57:58 UTC+0000                                 
0xfffffa800ed1b5d0 svchost.exe            1200    484     11      266      0      0 2020-11-06 08:57:58 UTC+0000                                 
0xfffffa800ed7e9c0 svchost.exe            1280    484      8      160      0      0 2020-11-06 08:57:58 UTC+0000                                 
0xfffffa800eef7b10 taskhost.exe           1896    484      9      198      1      0 2020-11-06 08:58:02 UTC+0000                                 
0xfffffa800ef16570 dwm.exe                1952    812      3       72      1      0 2020-11-06 08:58:02 UTC+0000                                 
0xfffffa800ef1e4b0 explorer.exe           1964   1944     19      729      1      0 2020-11-06 08:58:02 UTC+0000                                 
0xfffffa800ef2bb10 StikyNot.exe            120   1964      8      137      1      0 2020-11-06 08:58:03 UTC+0000                                 
0xfffffa800eea1b10 SearchIndexer.         1684    484     11      609      0      0 2020-11-06 08:58:03 UTC+0000                                 
0xfffffa800f049550 firefox.exe             824   1848     60     1079      1      1 2020-11-06 08:58:10 UTC+0000                                 
0xfffffa800f04eb10 firefox.exe            2076    824      9      277      1      1 2020-11-06 08:58:11 UTC+0000                                 
0xfffffa800f162b10 firefox.exe            2208    824     18      306      1      1 2020-11-06 08:58:12 UTC+0000                                 
0xfffffa800f199b10 firefox.exe            2424    824     18      310      1      1 2020-11-06 08:58:13 UTC+0000                                 
0xfffffa800f233b10 firefox.exe            2660    824     18      303      1      1 2020-11-06 08:58:15 UTC+0000                                 
0xfffffa800f231b10 firefox.exe            2668    824      0 --------      1      0 2020-11-06 08:58:15 UTC+0000   2020-11-06 08:58:15 UTC+0000  
0xfffffa800e300470 putty.exe              2900   1964      1       72      1      0 2020-11-06 08:58:19 UTC+0000                                 
0xfffffa801d3e5060 calc.exe               2916   1964      3       73      1      0 2020-11-06 08:58:21 UTC+0000                                 
0xfffffa800f201060 notepad.exe            2936   1964      1       57      1      0 2020-11-06 08:58:25 UTC+0000                                 
0xfffffa800e9efb10 firefox.exe            3056    824     18      307      1      1 2020-11-06 08:58:43 UTC+0000                                 
0xfffffa8014356810 firefox.exe            2744    824     35      478      1      1 2020-11-06 08:59:02 UTC+0000                                 
0xfffffa800ee99880 TrueCrypt.exe          2324   1964      4      249      1      1 2020-11-06 08:59:09 UTC+0000                                 
0xfffffa800f1636a0 svchost.exe             272    484     13      353      0      0 2020-11-06 08:59:59 UTC+0000                                 
0xfffffa800cec7b10 firefox.exe            1760    824     15      273      1      1 2020-11-06 09:00:18 UTC+0000                                 
0xfffffa800f196b10 firefox.exe            3344    824      5      157      1      1 2020-11-06 09:01:53 UTC+0000                                 
0xfffffa800cf5e060 WmiPrvSE.exe           3472    604      5      117      0      0 2020-11-06 09:01:58 UTC+0000  
```

We see firefox, notepad, calculator etc. but also TrueCrypt, which is a freeware utility that is sed for on-the-fly encryption. 

We want to take a look at this process further, and volatility has the truecryptsummary setting which we can use: 
`./volatility_2.6_lin64_standalone -f memory.dump --profile=Win7SP1x64 truecryptsummary`

```
Volatility Foundation Volatility Framework 2.6
Password             67Nj9kL11wQ.P-r5RmsDDx at offset 0xfffff88003a56e64
Process              TrueCrypt.exe at 0xfffffa800ee99880 pid 2324
Service              truecrypt state SERVICE_RUNNING
Kernel Module        truecrypt.sys at 0xfffff88003a1b000 - 0xfffff88003a5c000
Symbolic Link        Q: -> \Device\TrueCryptVolumeQ mounted 2020-11-06 08:59:50 UTC+0000
Symbolic Link        Q: -> \Device\TrueCryptVolumeQ mounted 2020-11-06 08:59:50 UTC+0000
Symbolic Link        Volume{dfc60663-1f7a-11eb-9bb6-000c29d34d53} -> \Device\TrueCryptVolumeQ mounted 2020-11-06 08:59:50 UTC+0000
File Object          \Device\TrueCryptVolumeQ\ at 0x3eec9220
File Object          \Device\TrueCryptVolumeQ\ at 0x3f1d5560
Driver               \Driver\truecrypt at 0x3ee454f0 range 0xfffff88003a1b000 - 0xfffff88003a5c000
Device               TrueCryptVolumeQ at 0xfffffa800f16f6f0 type FILE_DEVICE_DISK
Container            Path: \??\C:\Users\jack\Documents\secret
Device               TrueCrypt at 0xfffffa800dc452c0 type FILE_DEVICE_UNKNOWN

```

This shows that the user is probably Jack (as seen in the container file), and that the password is: 67Nj9kL11wQ.P-r5RmsDDx for the file in the path Users/jack/Documents/secret.

We can now use Truecrypt to decrypt the secret file with the obtained password. We can use cryptsetup for this. 

`sudo cryptsetup --type tcrypt open secret secret  `

Next, we mount: (we create a folder mountpoint first)
`sudo mount /dev/mapper/secret ./mountpoint  `

We then get a folder called Private, a textfile file.txt and a pdf file pdf.pdf. 

The file.txt contains: 
Visit this url: https://pastebin.com/XAKBsNEN

Which is the flag we need!
