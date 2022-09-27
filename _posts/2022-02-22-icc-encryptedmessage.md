---
layout: post
title: ICC - EncryptedMessage 
date: 2022-02-22 17:05 +0200
author: Arcee
tags: challenge crypto 
categories: [ICC]
---
Challenge of the training platform for ICC.

## Description: 
The attackers have left behind a file in a museum server. It seems to be encrypted and accompanied with a RSA public key.
You know that RSA encryption is very strong and you need billions of most powerful servers on Earth to crack it.
But closer look at the encrypted file and public key reveals some very promising information - used encryption is not very strong.

You need to find two prime numbers from modulus of public key and then create private key, which decrypts the encrypted message.

Question: Find the hidden message from the encrypted document. Post the URL as an answer

## Message file
The message file is a zip file (according to `file message`), so we unzip it. Which gives us files message1.enc, message2.enc, message3.enc, message4.enc and public.crt. Where the first four are encrypted message files, and the fifth is the public key.

### public.crt file
Public key is very short (so it should probably be very easy to break)
```
-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMfbt8SyV45gzYmKeBnqw1DFg5eeopzR
NDwHYmgN1D8PAgMBAAE=
-----END PUBLIC KEY-----
```

RsaCtfTool gives as private key: `python3 RsaCtfTool.py --publickey public.crt --private` and we save the private key to a file called `key.priv`

```
-----BEGIN RSA PRIVATE KEY-----
MIGqAgEAAiEAx9u3xLJXjmDNiYp4GerDUMWDl56inNE0PAdiaA3UPw8CAwEAAQIg
GB/+q+C3TvmCdhLf8ojzMIs33XDlOVNl20EcXk+ElIkCEQDdRIngbugTUscbXhmJ
p9+tAhEA5zrboAVb/Y8tLk1k8goBKwIQRwoHPkA9UF7mP/ohNtnn7QIQHHvCnCas
20Is1ZxRCAO1ewIRAJzJ2HpB1lIzXQpHWSsgc1o=
-----END RSA PRIVATE KEY-----
```

We can now  use openssl to decrypt the encrypted message files using the following command: 

`openssl rsautl -decrypt -inkey key.priv -in messageX.enc`

Which gives us the following values:
VG8gZ2V0IHRoZSBmbGFnL
CB2aXNpdCB0aGlzIFVSTD
ogaHh4cHM6Ly9wYXN0ZWJ
pbi5jb20vTnQ4czBTWXM

If we put this in boxentriq cipher analyzer it shows that this is most likely base64, so we use cyberchef to decode the base64 text giving us: 

" To get the flag, visit this URL: https://pastebin.com/Nt8s0SYs"

So the flag is: https://pastebin.com/Nt8s0SYs.

