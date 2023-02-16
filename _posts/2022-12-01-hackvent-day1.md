---
layout: post
title: HACKvent - Day 1 (HV22.01)
date: 2022-12-01 16:08 +0100
author: Arcee
tags: hacking-lab easy fun
categories: [HackingLab, Challenge, Fun]
---
First challenge in HACKvent 2022

## Challenge:

**Name**: QR means quick reactions, right? 

**Category**: Fun

**Solve date**: 1-12-2022

**Challenge Description**: Santa's brother Father Musk just bought out a new decoration factory. He sacked all the developers and tried making his own QR code generator but something seems off with it. Can you try and see what he's done wrong?

## Investigation
We get a gif containing a lot of QRcodes in christmas baubles. If we use a tool that extracts all frames from GIFS we get all the different QR codes, which spell out the flag. 
