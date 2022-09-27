---
layout: post
title: ICC - EasyCrypt 
date: 2022-02-24 15:48 +0200
author: Arcee
tags: challenge crypto icc 
categories: [ICC]
---
Challenge of the training platform for ICC.
 
## Description: 
Ransomware made by a very low level adversary, written in PHP, was used to encrypt files inside internal network.
We hope that you can help and decrypt our files to get back valuable information.

**Question**  
Analyze the encryption code and decrypt the file to reveal the flag  

#### Solution:
We get an encrypted file with the .payusransom extension, and the ransomware php code. When looking at the PHP code we can see that this can be easily reversed. What we need to do on the file is: 

``` 
<?php
$ALGO	 	= 'AES-256-CBC';
$IV 		= openssl_random_pseudo_bytes(openssl_cipher_iv_length($ALGO));
$date 		= new DateTime();
$password 	= $date->getTimestamp();
if (isset($_POST) && isset($_FILES['file'])){ 
## IsSet is used to call the PHP function. So if a POST request is made, the following php code is executed. 
## _FILES is the array of items uplodaed to the script via the HTTP POST method. 	
    $file      = isset($_FILES['file']);
    $content   = '';
    $content   = file_get_contents($_FILES['file']['tmp_name']);
    $filename  = $_FILES['file']['name'];
    $content   = openssl_encrypt($content, $ALGO, $password, 0, $IV);
    $content   .= $IV;
    $filename  = $password . '.payusransom';
    header("Pragma: public");
    header("Pragma: no-cache");
    header("Cache-Control: no-store, no-cache, must-revalidate, max-age=0");
    header("Cache-Control: post-check=0, pre-check=0", false);
    header("Expires: 0");
    header("Content-Type: application/octet-stream");
    header("Content-Disposition: attachment; filename=\"" . $filename . "\";");
    $size = strlen($content);
    header("Content-Length: " . $size);
    echo $content;
    die;
}
?>
```

1. Open file (password.payusransom)    
Where the password is the getTimestamp of the date.

2. Remove the IV from the file content, which is appended at its back.   
We have an IV of length: `openssl_cipher_iv_length('AES-256-CBC');` = 16. So the final 16 characters of the content of the encrypted file is the IV.  

3.  Decrypt using the openssl_decrypt method. As we know all the variables we need. We have the IV, we know the algorithm, and the password is the filename. 

The php script for this is: 
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

When we execute this we get the flag in the decrypted file!
