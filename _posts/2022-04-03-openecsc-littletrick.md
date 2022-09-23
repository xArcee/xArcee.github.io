---
layout: post
title: OpenECSC - Little Trick
date: 2022-04-03 15:51 +0200
author: Danique
tags: ctf openecsc rev
categories: [CTF, Reverse Engineering]
---
Easy challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 3-4-2022.

## Challenge Description
Please start the docker service from RESOURCES. Download the executable and find the missing XXXXXXXX and complete the UUID.

## Solution
We get a littletrick file which is an ELF executable. When we run it it asks us for a password, which we do not know. When entering a random password it gives us the message 'wrong password, no flag. sorry'.

```
$ ./littletrick    
Password? e
Wrong password, no flag. Sorry
```

We can open the file in Ghidra and see if we can change some code to make it continue with any password.

When we load the file in Ghidra we see a main funtion that looks like it checks the password in memory with the password given by the user. If the memcmp of these two is equal to 0 (meaning they are equal) it is continued to a decrypt function which gives us the flag result.

We can change this if statement in line 102 from an == to an != by changing the assembly instruction from JNZ (jump not zero) to JZ (jump if zero), so the jump is always executed if the password the user gives is not equal to the password in the file. We change the expression and then save the patched file with the SavePatch script in Ghidra to create a new executable: littletrick_new.

```
$ ./littletrick_new 
Password? ee
03283fa2
```

When we now run littletrick_2 and enter any random password it gives us the flag:
So the final flag is:
03283fa2-f037-4fe3-a582-15a933d7d3d2


