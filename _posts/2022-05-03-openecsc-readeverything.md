---
layout: post
title: OpenECSC - Read Everything
date: 2022-05-03 15:51 +0200
author: Danique
tags: ctf openecsc pwn rev crypto
categories: [CTF, Binary Exploitation, Reverse Engineering, Crypto]
---
Easy challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 3-5-2022.

## Challenge Description
Start the service from RESOURCES. The docker is offering two ports (web and socket port).

port 80
port 1342
Exploit port 1342.

## Solution
We can download the server file from the ip:80 url. Which when executed asks for a length of a password, after which you have to type the password. Analyzing the file in Ghidra does not show any flag etc. We do see that when the password is 0 characters it automatically continues and no password is asked.


Reading the challenge text it shows that we should exploit port 1342. So when accessing 152.96.2.7:1342, we get an internet page with the same server text. Only it now also shows a flag! As the password is set with 0 characters.

```
How long password would you like to set? Ok, please enter your 0-character password: Retype for confirmation: Saved.
8e05f432-6dfd-441d-93b9-a018037c5eb9
```
