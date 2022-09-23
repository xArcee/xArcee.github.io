---
layout: post
title: OpenECSC - Encoded File Challenge
date: 2022-05-03 15:51 +0200
author: Danique
tags: ctf openecsc misc
categories: [CTF, MISC]
---
Easy challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 3-5-2022.

## Challenge Description
Decode the encoded file and find the flag.

## Solution
We get an encoded.txt file, that if we run the 'file' command on shows that it is a gzip compressed file. Therefore we change the file extension from .txt to .gz and unzip the file using `gunzip encoded.gz`.

This gives us a file called encoded that contains csv text. We see that the first column contains the numbers 1 to 29 and the second column is either of the value 20 or the value 4 (depending on what is in the third column). So we should probably sort the text in the csv file by the first column and then convert the hex from the ordered third columns into ascii text. The second column shows the size of the hex. 

When we convert all hex to ascii we get a base-64 encoded text, which we add to the file `base64-encoded.txt`. We decode this using `base64 -d base64-encoded.txt > result`

This file `result` is a bzip2 compressed data, so we change the file extension to `result.bz2` and extract the content using `bunzip2 result.bz2`, which finally gives us an ASCII text file which reads the flag!


