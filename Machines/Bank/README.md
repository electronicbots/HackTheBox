# Bank-HTB
Bank is a Linux machine that rated as easy. So let's start :)

# Recon
I started by runing nmap scan on the machine and found 3 open ports.
```
root@kali:~/Desktop# nmap -sC -sV 10.10.10.29
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-09 13:58 EDT
Nmap scan report for bank.htb (10.10.10.29)
Host is up (0.027s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-title: HTB Bank - Login
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.48 seconds
```
First thing to do is add ```bank.htb``` to ```/etc/hosts```. add this line in the file ```10.10.10.29     bank.htb```
Now if we browse to bank.htb we will find a login page.
![image1](url)
## ffuf
As always I like to use ffuf to fuze web directories
```
root@kali:~/Desktop/tools/Web/FFUF# ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://bank.htb/FUZZ.php

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://bank.htb/FUZZ.php
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

                        [Status: 403, Size: 279, Words: 21, Lines: 11]
# Copyright 2007 James Fisher [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Suite 300, San Francisco, California, 94105, USA. [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# or send a letter to Creative Commons, 171 Second Street,  [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/  [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
login                   [Status: 200, Size: 1974, Words: 595, Lines: 52]
# Priority ordered case sensative list, where entries were found  [Status: 302, Size: 7322, Words: 3793, Lines: 189]
support                 [Status: 302, Size: 3291, Words: 784, Lines: 84]
# This work is licensed under the Creative Commons  [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# on atleast 2 different hosts [Status: 302, Size: 7322, Words: 3793, Lines: 189]
index                   [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# directory-list-2.3-medium.txt [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Attribution-Share Alike 3.0 License. To view a copy of this  [Status: 302, Size: 7322, Words: 3793, Lines: 189]
logout                  [Status: 302, Size: 0, Words: 1, Lines: 1]
                        [Status: 403, Size: 279, Words: 21, Lines: 11]
:: Progress: [220560/220560] :: Job [1/1] :: 1102 req/sec :: Duration: [0:03:20] :: Errors: 0 ::
```
And we found ```support.php```
