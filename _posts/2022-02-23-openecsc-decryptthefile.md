---
layout: post
title: OpenECSC - Decrypt the File
date: 2022-02-23 15:51 +0200
author: Danique
tags: ctf openecsc crypto
categories: [CTF, Crypto]
---
Medium challenge in the OpenECSC online environment, round 1, on HackingLab. Solved on 23-02-2022.

## Challenge Description
Ransomware made by a very low level adversary, written in PHP, was used to encrypt files inside the internal network. We hope that you can help and decrypt our files to get back valuable information.

## Solution
We get an encrypted file with the .payusransom extension, and the ransomware php code. When looking at the PHP code we can see that this can be easily reversed. 
IsSet is used to call the PHP function. So if a POST request is made, the given php code is executed. 
_FILES is the array of items uploaded to the script via the HTTP POST method. So we need to reverse the php code to get the decrypted version of the file.
What we need to do on the file is: 


1. Open file (password.payusransom)    
Where the password is the getTimestamp of the date.

2. Remove the IV from the file content, which is appended at its back.   
We have an IV of length: `openssl_cipher_iv_length('AES-256-CBC');` which is equal to 16. So the final 16 characters of the content of the encrypted file is the IV.  

3.  Decrypt using the openssl_decrypt method. As we know all the variables we need. We have the IV, we know the algorithm, and the password is the filename. 

I have written the following php script to decrypt the contents of the file:
```
<?php

$ALGO	 	= 'AES-256-CBC';
$password = '1626253572';

# Open the image:
$content = file_get_contents('1626253572.payusransom');

#Get the IV:
$IV = substr($content, -16); #Is in binary, can be converted to hex with bin2hex

#Get actual encrypted content
$encr_content = str_replace($IV, "", $content);

# Decrypt the encrypted content:
$decr_content = openssl_decrypt($encr_content, $ALGO, $password, 0, $IV);

print($decr_content);

?>
```

When we execute this we get the following contents of the file: 

The Flag is: 8932a828-2127-4512-9792-e3c07da240c0


