---
layout: post
title: OpenECSC - Greeting Card
date: 2022-06-20 15:51 +0200
author: Danique
tags: ctf openecsc forensics
categories: [CTF, Forensics]
---
Easy challenge in the OpenECSC online environment, round 3, on HackingLab. Solved on 20-6-2022.

## Challenge Description
A friend of yours asks to take a look at a strange document. Find answers to the questions below
1. What is written in the message?
2. What KDF was used as part of the encryption algorithm for the message?
3. What is the IV for decrypting the final flag? HEXPRINT
5. What is the IP:PORT of the C&C server?
6. What is the final flag?

## Solution

1. What is written in the message?
Message: U2FsdGVkX1/WWNSDJ9yz90xW5DAFJ2iTorGmSweqB8O/iSqBjhLBFe/ls2m+iWgc
Which is in base64, so using cyberchef to decrypt gives:

`Salted__ÖXÔ.'Ü³÷LVä0.'h.¢±¦K.ª.Ã¿.*...Á.ïå³i¾.h.`

Which is salted, so we need to do some openssl decryption.

Opening the .odt file as an archive and looking at the content.xml file gives us also "Encryption algorithm: AES-128, Key Derivation function:PBKDF2"
```
$ exiftool image2.jpeg            
ExifTool Version Number         : 12.41
...
Document Notes                  : Password is blackRh!no48
...
```
This shows Password is blackRh!no48'. So we can now decrypt the salted string using openssl.
The text from the file is copied to originaltext.txt

We first decode base64: `base64 -d originaltext.txt > salted.txt`
and then we use openssl: `openssl aes-128-cbc -d -in salted.txt -out output.txt -k "blackRh\!no48" -pdkdf2`
Which gives us: `The answer is fc72b8656ffd25`

2. What KDF was used as part of the encryption algorithm for the message?
We know that PBKDF2 is used as the KDF, from the previous information.

3. What is the IV for decrypting the final flag? Hexprint
Unzipping comment.zip with the password from question 1 gives a comment.xml file containing the IV: 4185115006824300

The text gives that it should be in hexprint, so we use cyberchef to change to hex:
34313835313135303036383234333030

4. What is the IP:Port of the C&C Server?
Spectogram: W2ZkMDE6MDoyNTU6ODc4OjJhMDk6OmJlZWY6MTgzXTozMTMzNw==
Which is again base64, so we decode:
`echo W2ZkMDE6MDoyNTU6ODc4OjJhMDk6OmJlZWY6MTgzXTozMTMzNw== | base64 -d` which gives: `[fd01:0:255:878:2a09::beef:183]:31337`


5. What is the final flag?
Comment.xml gives us all the information abuot the encryption, and meta.xml has a hexdump comment that we can use to decrypt.


We use CyberChef to first use 'From Hexdump' and then 'AES Decrypt' using the provided key and IV. Which gives:
```
The final flag is:

	35948374-f3a8-4b62-9c2d-7dbf200ec690

Pass it to the script as second argument!
```
