---
layout: post
title: ICC - Rootkit
date: 2022-02-24 17:10 +0200
author: Arcee
tags: challenge rev 
categories: [ICC]
---
Challenge of the training platform for ICC.

## Description: 
Cyber police recovered rootkit that was used to steal 1000 Bitcoins from Online trading platform.
They are asking for help to retrieve hidden port that was used to access compromised server.
Only clue they can give You is that this rootkit hold a different functions but only one can be used as mentioned backdoor.
This will require some skills in reverse engineering and instruction reading but stakes are too high to fail.

Question: Can you help Cyber Police to find a critical clue in investigation?

Files:
http://workshop-static.cybexer.io/rootkit.so

## Rootkit file
File information: (running `file rootkit.so`):   
rootkit.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=6517a857d360de3ec4a99ade5c8f481dc20047ce, not stripped

The `nm` command is used to list the symbols from the target program. By using this we cna get to know the local and library functions and also the global variables used. It cannot work on a program that is stripped, but as this program is not stripped we can use it:

```
nm rootkit.so | grep '  T ' 
```

We only want to view the code sections, so we grep for the 'T'  symbols. Which gives the following functions:

0000000000000d0b T accept
0000000000000e00 T falsify_tcp
0000000000001164 T _fini
0000000000000ef0 T fopen
0000000000001029 T fopen64
0000000000000a60 T _init
0000000000000c9a T readdir

From this list the functions falsify_tcp and accept look interesting, as the rootkit is used to install a backdoor into the system. 

We will first take a look at the falsify_tcp function by loading the file into the IDA disassembler. 

### falsify_tcp function
This function looks like it moves a lot of variables form certain registers to others, not much else

### accept function
Here we see something interesting, the htons function is called, which is used to make sure that numbers are stored in memory in network byte order (with the most significant byte first). This has to do with the network protocols used, which require the transmitted packets to use network byte order. So it swaps the endianness of a short. 

The line above this states the comment hostshort, which is the port we need! The hex value 4E09h (the h denotes that it is a hex value) is moved into the edi register. 

So the hidden port is 4E09, which is in decimal: 19977
So the hidden port is 19977.
