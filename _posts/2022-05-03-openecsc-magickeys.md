---
layout: post
title: OpenECSC - Magic Keys
date: 2022-05-03 15:51 +0200
author: Danique
tags: ctf openecsc crypto
categories: [CTF, Crypto]
---
Easy challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 3-5-2022.

## Challenge Description
Please start the magic service from RESOURCES. Use the hints below to get KEY1 and KEY2 from the service. The service will also disclose the missing PADDING information.

## Solution
We can connect to the server using netcat. The server gives us the padding, and we can choose between the two keys and decrypt a ciphertext message. Looking at the code provided we can see that AES encryption in CBC mode is used, however the key is used as an IV. We can exploit this!
```
Here you have a hint! The padding for the flag is: 58fa
```

To exploit this we need a ciphertext of 3 blocks long, of which the first and third block are the same and the second block is all zeros.

So we use the following message `aaaaaaaaaaaaaaaa000000000000000aaaaaaaaaaaaaaaa`, which in hex is:
`616161616161616161616161616161610000000000000000000000000000000061616161616161616161616161616161`

We first use the decrypt function for key 1. which gives us the following:
```
Block 1: 0b29ca528f77800d2ba995405515d6e8
Block 2: 9b56a9d89335c29385188e8b5753f394
Block 3: 3b1ef964ea13e238069ba5783738e289
```

We now XOR the first and the third block, which gives us our key!
```
xor: 30373336656462352d323038622d3461
```

Which converted to ascii gives us the first key: 0736edb5-208b-4a
Next we do the same for key 2:
```
Block 1: c0690e46f92efabe4033992e3ff13599
Block 2: 7a5452783b0ccb772dd17b1ae0f2239f
Block 3: f859237ecd16c8937450af165bc005a9
```

And again XOR the first and third block:
```
xor: 38302d383438322d3463363864313030
```

Which we convert to ascii and gives us the second key: 80-8482-4c68d100


So the final key is: KEY1+KEY2+PADDING
`0736edb5-208b-4a80-8482-4c68d10058fa`
