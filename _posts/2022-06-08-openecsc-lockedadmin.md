---
layout: post
title: OpenECSC - Locked Admin
date: 2022-06-08 15:51 +0200
author: Danique
tags: ctf openecsc forensics pentest
categories: [CTF, Forensics, Penetration Testing]
---
Medium challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 8-6-2022.

## Challenge Description
This time you must investigate a recent attack on a web server. Attackers have dropped some password-protected site there. Your friends from Incident Response Department managed to get a network capture file from one of the attackers' computers.

## Solution
We get a pcap with USB interrupts in packets, which are the keystrokes of a keyboard sent to the PC. What we specifically need from these packets are the Leftover Capture Data fields. Which we can get using the following command:
```
tshark -r network.pcapng -T fields -e usb.capdata | tr -d : | tr -s "\n" > output.txt
```

We now have all the Leftover Capture Data saved in the output.txt file. We can then use a script found online (and slightly modified to use the correct translations (which were found here: https://www.usb.org/sites/default/files/documents/hut1_12v2.pdf)

Script:
```
#!/usr/bin/python
# coding: utf-8
from __future__ import print_function
import sys,os

#declare -A lcasekey
lcasekey = {}
#declare -A ucasekey
ucasekey = {}

#associate USB HID scan codes with keys
#ex: key 4  can be both "a" and "A", depending on if SHIFT is held down
 # Insert very long list of lcasekey and ucasekey here which we use, but cant put in the writeup because otherwise its too long

#make sure filename to open has been provided
if len(sys.argv) == 2:
	keycodes = open(sys.argv[1])
	for line in keycodes:
		#dump line to bytearray
		bytesArray = bytearray.fromhex(line.strip())
		#see if we have a key code
		val = int(bytesArray[2])
		if val > 3 and val < 100:
			#see if left shift or right shift was held down
			if bytesArray[0] == 0x02 or bytesArray[0] == 0x20 :
				print(ucasekey[int(bytesArray[2])], end=''),  #single line output
				#print(ucasekey[int(bytesArray[2])])            #newline output
			else:
				print(lcasekey[int(bytesArray[2])], end=''),  #single line output
				#print(lcasekey[int(bytesArray[2])])            #newline output
else:
    print("USAGE: python %s [filename]" % os.path.basename(__file__))
```

Now we can execute the code by using `python decode.py output.txt` which gives us:
```
adminTABpassBACKSPACEBACKSPACEsswordENTERicscadminTAB0nc3HomeDelForwardOEnd@g@inLeftArrowBACKSPACE1RightArrowW3needLeftArrowBACKSPACEBACKSPACE33End@p@LeftArrowBACKSPACEPRightArrowsswoBACKSPACE0$d!BACKSPACE_BACKSPACE
```
Which if we replay using our own keyboard gives us:
admin	password
icscadmin Onc3@g@1nW3n33d@P@ssw0$d

Which we can use to log in, and it gives us the flag!
