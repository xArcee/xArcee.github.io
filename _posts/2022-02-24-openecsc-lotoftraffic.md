---
layout: post
title: OpenECSC - Lot of Traffic
date: 2022-02-24 15:51 +0200
author: Danique
tags: ctf openecsc forensics
categories: [CTF, Forensics]
---
Easy challenge in the OpenECSC online environment, round 1, on HackingLab. Solved on 24-02-2022.

## Challenge Description
Some time ago a former system administrator was debugging some networking issues and made a wireless traffic dump. Try to crack the wireless password and explore the decrypted traffic! You should be able to find something interesting from there.

Goal: Find the flag in the pcap dump, the flag format is a string. 

## Solution
We get a wireless.pcap file, which if we inspect is WEP encrypted wireless traffic. Therefore, before we can see any of the actual traffic, we need to first decrypt the traffic we have now. This can be done using aircrack-ng to try and crack the password. 

Initial running of `aircrack-ng wireless.pcap` does not result in anything, so we try to bruteforce with `aircrack-ng -K wireless.pcap` which does give us a key!
```
KEY FOUND! [ 0F:A4:CA:1E:66 ] 
```

Now we can use this key in wireshark to decrypt the traffic, which gives a lot of MPEG TS protocol packets. 
MPEG TS is a standard digital container format for transmission and storage of audio, video and PSIP data. So there is probably some audio or video file being sent in the traffic. 

We use the `mpeg_dump.lua`-tool for wireshark to extract the MPEG stream: wiki.wireshark.org/mpeg_dump.lua.
We export the stream as output.ts and then view it in VLC. This shows us an image of something along the likes of Alice in Wonderland and the flag is in the image!


