---
layout: post
title: OpenECSC - Docker Forensics
date: 2022-04-30 15:51 +0200
author: Danique
tags: ctf openecsc forensics
categories: [CTF, Forensics]
---
Novice challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 30-4-2022.

## Challenge Description
Find the flag.

## Solution
We get a page with a QR code asking us to enter a session ID. Initial idea is that you probably need to be quick enough to enter the value of the qrcode in the box so you get the flag. Therefore we write a script to do this quickly.

```
## Importing Necessary Modules
import cv2
import requests # to get image from the web
import shutil # to save it locally

r = requests.get("https://fd8a3235-1153-4c72-9abf-0ba314dcfc9a.idocker.vuln.land/", headers={"Cookie" : "PHPSESSID=4dfjhec5sg4squiis0iluj8h72"})

## Set up the image URL and filename
image_url = "https://fd8a3235-1153-4c72-9abf-0ba314dcfc9a.idocker.vuln.land/gen.php?s=qrh&d="
filename = "qrcode.png"

# Open the url image, set stream to True, this will return the stream content.
r = requests.get(image_url, stream = True, cookies={"PHPSESSID" : "4dfjhec5sg4squiis0iluj8h72"})
    
# Open a local file with wb ( write binary ) permission.
with open(filename,'wb') as f:
	shutil.copyfileobj(r.raw, f)

#filename = "qrcode.png"
image = cv2.imread(filename)
detector = cv2.QRCodeDetector()
data, vertices_array, binary_qrcode = detector.detectAndDecode(image)
if vertices_array is not None:
	print("QRCode data: ")
	print(data)

r2 = requests.post("https://fd8a3235-1153-4c72-9abf-0ba314dcfc9a.idocker.vuln.land/loader.php", data={"login" : data, "submit" : "Login"}, cookies={"PHPSESSID" : "4dfjhec5sg4squiis0iluj8h72"})

print(r2.text)
```
We first download the QRcode image, after which we decode it using cv2 and then send the decoded QRcode value as a post request to the correct url. It is important to add all the data and header content in the requests as otherwise you will get an error and not the flag. Code is not that neat (hardcoded URLs), but does the job.



