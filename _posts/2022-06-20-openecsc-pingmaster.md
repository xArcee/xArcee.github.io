---
layout: post
title: OpenECSC - Pingmaster
date: 2022-06-20 15:51 +0200
author: Danique
tags: ctf openecsc web
categories: [CTF, Web]
---
Easy challenge in the OpenECSC online environment, round 3, on HackingLab. Solved on 20-6-2022.

## Challenge Description
Please start the ping service from RESOURCES. It offers a web-based ping utility that awards you with a flag when you are able to ping a private IPv4 address 192.168.99.7. Public IP's are allowed (but firewalled).

## Solution
You are able to ping public IPS, but not private ips. However, if you change the IP to UINT it still pings!

So if we want to ping 192.168.99.7, we use `http://www.aboutmyip.com/AboutMyXApp/IP2Integer.jsp?ipAddress=192.168.99.7` which gives us an Integer: 3232260871.
Which if we enter this into the form, gives us the flag!
```
Regex bypassed! Flag is: 78eb935c-9901-4602-af88-ec9f4dd2050c
```
