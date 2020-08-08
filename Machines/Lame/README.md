# Lame-HTB

Lame is a Linux machine that rated as easy. It was the first box that got released on hackthebox. So let's start :)

# Recon

I started by runing nmap scan on the machine and found 5 open ports.

```nmap -sC -sV 10.10.10.3```

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-08 16:41 EDT
Nmap scan report for 10.10.10.3
Host is up (0.099s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.19
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h02m41s, deviation: 2h49m43s, median: 2m40s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-08-08T16:44:10-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.57 seconds
```
## FTP
I tried Anonymous Login but didn't find anything. vsFTPd 2.3.4 is an old version so I checked searchsploit and found an exploit for it
```
root@kali:~/Desktop/HTB/Lame# searchsploit vsFTPd 2.3.4
--------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                   |  Path
--------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                           | unix/remote/17491.rb
--------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
## SMB
I tried using ```smbmap``` and found that I have access to one share without needing for any credentials

```
root@kali:~/Desktop/HTB/Lame# smbmap -H 10.10.10.3
[+] IP: 10.10.10.3:445  Name: 10.10.10.3                                        
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        tmp                                                     READ, WRITE     oh noes!
        opt                                                     NO ACCESS
        IPC$                                                    NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$                                                  NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))        
```
So let's try access the shared folder using ```smbclient```:

```
root@kali:~/Desktop/HTB/Lame# smbclient -N //10.10.10.3/tmp
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Aug  8 18:54:04 2020
  ..                                 DR        0  Sun May 20 14:36:12 2012
  5143.jsvc_up                        R        0  Sat Aug  8 16:20:12 2020
  .ICE-unix                          DH        0  Sat Aug  8 16:19:08 2020
  .X11-unix                          DH        0  Sat Aug  8 16:19:34 2020
  .X0-lock                           HR       11  Sat Aug  8 16:19:34 2020

                7282168 blocks of size 1024. 5678780 blocks available
smb: \>
```
And there is nothing intrested. Let's try to search for exploit for samba 3.0

```
root@kali:~/Desktop/HTB/Lame# searchsploit samba 3.0
----------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                             |  Path
----------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 (OSX) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                       | osx/remote/16875.rb
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                     | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                           | unix/remote/16320.rb
Samba 3.0.21 < 3.0.24 - LSA trans names Heap Overflow (Metasploit)                                         | linux/remote/9950.rb
Samba 3.0.24 (Linux) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                     | linux/remote/16859.rb
Samba 3.0.24 (Solaris) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                   | solaris/remote/16329.rb
Samba 3.0.27a - 'send_mailslot()' Remote Buffer Overflow                                                   | linux/dos/4732.c
Samba 3.0.29 (Client) - 'receive_smb_raw()' Buffer Overflow (PoC)                                          | multiple/dos/5712.pl
Samba 3.0.4 - SWAT Authorisation Buffer Overflow                                                           | linux/remote/364.pl
Samba < 3.0.20 - Remote Heap Overflow                                                                      | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                      | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                              | linux_x86/dos/36741.py
----------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
