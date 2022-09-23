---
layout: post
title: OpenECSC - Clickergame
date: 2022-05-03 15:51 +0200
author: Danique
tags: ctf openecsc mobile
categories: [CTF, Android]
---
Medium challenge in the OpenECSC online environment, round 2, on HackingLab. Solved on 3-5-2022.

## Challenge Description
This challenge is about Android APK analysis. The flag is hidden in the APK. Can you find it?

## Solution
We use Genymotion with a Google Pixel XL device. First we place the .apk file in the folder where the adb tool is also located in Windows (Program Files\Genymobile\tools) and then run the command `.\adb install ClickerGame.apk` to install the app on the Pixel device.

When we open the clickergame we see that in order to 'buy' the flag we need 100000 coins, and for each click you get 1 coin. This will take a very long time to manually do, so we try to make some changes to the code.


We open the file with Bytecode Viewer (administrator 'java -jar Bytecode-Viewer-2.11.2.jar). We can find all the code in the ctf\clickergame folder, such as the MainActivity. In the MainActivity$3 class we can see the onClick for the flag button. Where this$0.coins >= 100000L is needed before the flag is printed.

We now use apktool to open all the files (`apktool d ClickerGame.apk`) which creates a new directory ClickerGame. We can find all the app files in the smali\ctf\clickergame folder. We know that we need to edit the MainActivity$3.smali file to change the input for the flag. On line 46 we see the constant for the amount needed for the flag (0x186a0), so we change this to 0x1 so we only need 1 coin.

Now we need to rebuild the apk. We do this with apktool by running `apktool b ClickerGame`. Next we need to uninstall the old app on the Genymotion phone and use adb to install the new version of the app. We can find the new apk in the dist folder in the ClickerGame folder, however we can't install this immediately as this will give us an error. So we need to make some changes. We need to sign the file.

We add the following code to a sign-apk.bat file:

```
keytool -genkeypair -v -keystore key.keystore -alias publishingdoc -keyalg RSA -keysize 2048 -validity 10000

jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ./key.keystore %1 publishingdoc
```
Next, we run `sign-apk.bat ClickerGame.apk` with the password 123123. This will sign the apk. Now we can use `.\adb install ClickerGame.apk` which now does work. Then we open the game and click once and then buy the flag, which gives us the flag!: ctf{6A58CDCFCFE5C3AA41E01A5908BD9F8D}
