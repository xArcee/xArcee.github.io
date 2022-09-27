---
layout: post
title: ICC - Unknown Device
date: 2022-02-24 17:40 +0200
author: Arcee
tags: challenge forensics 
categories: [ICC]
---
Challenge of the training platform for ICC.

## Description: 
One day network administrator discovered a unknown device in his office network.
He was able to capture network traffic between unknown device and public IP.
He discovered that unknown device is using secure and unsecure communication protocols. 

**Question**
Can you help your network administrator investigate network traffic on a suspicious device, and discover the flag from unsecure communication? 

File: https://static.icsc.cybexer.io/unknown-device.pcap

## Enumeration in pcap file
There are only 4 endpoint IPv4 IPs used in the entire pcap:

- 8.8.8.8 (alleen ICMP van/naar 192.168.1.254)
- 100.100.100.1 (alleen communication van/naar 192.168.11.1, ISAKMP, ESP en PPP?)
- 192.168.1.254 (alleen  ICMP van/naar 8.8.8.8, niet veel packets)
- 192.168.11.1  (alleen van/naar  100.100.100.1, ISAKMP, ESP en PPP?)

#### IPSec ISAKMP/ESP
ISAKMP and ESP (used by 100.100.100.1 and 192.168.11.1) are IPSex protocols. A tunnel is created with ISAKMP (Internet Security Association and Key Management Protocol) through which parameters for ESP (Encapsulating Security Payload) are negotiated.   
-> https://celaldogan2010.medium.com/decrypting-ipsec-protocols-isakmp-and-esp-with-wireshark-d484a5a93991

So if we want to get the data being communicated we need to decrypt the ESP packets. However, this can only be done by using a key, which we do not know. (https://wiki.wireshark.org/ESP_Preferences)

So the IPSec (ISAKMP/ESP) communication is probably the secure protocol that the unknown device is using. 

#### PPP / PPTP
PPTP is Point-to-Point Tunneling Protocol is an obsolete method for implementing Virtual Private Networks and it has many well known security issues (https://en.wikipedia.org/wiki/Point-to-Point_Tunneling_Protocol)

This could be our in as this is unsafe. When we look at the PPTP or GRE packets (filter on 'pptp or gre') we see that there are 2 PPTP startup blocks, so there are two sessions. In the second session we also see packets with the info "Compressed Data", which is probably what we need. So we only need to look at this one session, as the first session only contains authentication.  
We can filter only the second session using the filter 'pptp or gre and frame.number > 100'.

Different PPP protocols used: 
- PPP Comp
- PPP IPCP
- PPP LCP
- PPP MPLSCP

https://datatracker.ietf.org/doc/html/rfc1661 -> PPP information

"PPP provides a standard method for transporting multi-protocol datagrams over point-to-point links. It is compromised of three main components: 
1. A method for encapsulating the multi-protocol datagrams
2. A Link Control Protocol (LCP) for establishing, configuring and testing the data-link connection
3. A family of Network Control Protocols (NCPs) for establishing and configuring different network layer protocols."

##### CHAP
MS-CHAP-V2  (https://www.rfc-editor.org/rfc/rfc2759.html)  
**Challenge packet:**  
16-octet challenge Value field (random value)  
**Response packet:**
- 16 octets: Peer-Challenge    
Random number, used to calculate the NT-Response field. 
- 8 octets: Reserved, must be 0
- 24 octets: NT-Response
Encoded function of the password, the user name and the contents of the Peer-Challenge field and the received challenge as output by the GeneratedNTResponse()
- 1 octet: Flags    

**Success packet:**  
Message field contains a 42-octet authenticator response string and a printable message. 


Response packet value in text: 
96000cdbb6c254c4ea99a991b148b34e0000000000000000eeec9d8a2c9678bf641727ff5ff776331575f42916215cd100

Response packet value in binary:
10010110 00000000 00001100 11011011 10110110 11000010 01010100 11000100 11101010 10011001 10101001 10010001 10110001 01001000 10110011 01001110 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 11101110 11101100 10011101 10001010 00101100 10010110 01111000 10111111 01100100 00010111 00100111 11111111 01011111 11110111 01110110 00110011 00010101 01110101 11110100 00101001 00010110 00100001 01011100 11010001 00000000

So the peer challenge is:
10010110 00000000 00001100 11011011 10110110 11000010 01010100 11000100 11101010 10011001 10101001 10010001 10110001 01001000 10110011 01001110  
96000cdbb6c254c4ea99a991b148b34e

and the NT-Response:
11101110 11101100 10011101 10001010 00101100 10010110 01111000 10111111 01100100 00010111 00100111 11111111 01011111 11110111 01110110 00110011 00010101 01110101 11110100 00101001 00010110 00100001 01011100 11010001  
eeec9d8a2c9678bf641727ff5ff776331575f42916215cd1

#### Crack MS-CHAP-V2
John The Ripper can crack MS-CHAP-V2,
https://github.com/openwall/john/blob/bleeding-jumbo/src/MSCHAPv2_bs_fmt_plug.c

So we need to store the data we have in a specific format so John The Ripper can use it: 
```
USERNAME:::AUTHENTICATOR CHALLENGE:MSCHAPv2 RESPONSE:PEER CHALLENGE
USERNAME::DOMAIN:AUTHENTICATOR CHALLENGE:MSCHAPv2 RESPONSE:PEER CHALLENGE
DOMAIN\USERNAME:::AUTHENTICATOR CHALLENGE:MSCHAPv2 RESPONSE:PEER CHALLENGE
:::MSCHAPv2 CHALLENGE:MSCHAPv2 RESPONSE:
 ```
 
From the file we know the username (catch-me), the authenticator challenge (from the challenge packet), the NT-response and the peer challenge. 

Format this: 

catch-me:::41918206e33288b988722fb957328acc:::eeec9dt8a2c9678bf641727ff5ff776331575f42916215cd1:::96000cdbb6c254c4ea99a991b148b34e

We save this text to a file mschap.txt and use the rockyou.txt wordlist to find the password: 

`echo 'catch-me:::41918206e33288b988722fb957328acc:eeec9d8a2c9678bf641727ff5ff776331575f42916215cd1:96000cdbb6c254c4ea99a991b148b34e' > vpn.hash
`
(Moving Rockyou.txt to the folder to avoid errors)
`sudo john --format=mschapv2 --wordlist=rockyou.txt vpn.hash`  
Which gives:
```
Using default input encoding: UTF-8
Loaded 1 password hash (MSCHAPv2, C/R [MD4 DES (ESS MD5) 128/128 SSE2 4x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
!2moondreamer2!  (catch-me)     
1g 0:00:00:00 DONE (2022-02-24 14:48) 1.075g/s 15421Kp/s 15421Kc/s 15421KC/s !@#$5678..!$$lovemeorhateme$$!
Use the "--show --format=MSCHAPv2" options to display all of the cracked passwords reliably
Session completed.
```

So the password for the user catch-me is !2moondreamer2!

#### Actual data

We know that all the data is stored in the compressed data packets. 
When we look at the CCP (compression control protocol) packets we see that this has as type Microsoft PPE/PPC.

We now extract all the compressed datagrams, as well as the source and destination IPs using Tshark:
`tshark -r unknown-device.pcap -Y 'pptp or gre and frame.number>100 and comp_data' -T json -x | jq -r '.[]._source.layers | [ .ip."ip.src", .ip."ip.dst", .comp_data_raw[0] ] | @tsv ' | tee comp.dat` 

Now we can use a script from online that decrypts ppp, (ppp-decrypt.py) and execute it:

`python3 ppp-decrypt.py | tee decrypt.hex1`

Next, we convert the decrypt.hex1 file to a file that text2pcap understands. We convert to decrypt.pcap

`while read line; do echo $line | cut -d" " -f 2 | xxd -r -p | od -Ax -tx1 -v; echo; done < decrypt.hex1 > decrypt.hex2`

`text2pcap -l 101 decrypt.hex2 decrypt.pcap`

If we open this file in wireshark we now see a get request to the /secret.txt page, and if we look at the 200 OK response to that we find the flag in the message body: 

flag - 87d05a3ef6b00c260036f052c7303cb9 
