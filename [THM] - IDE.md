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

```bash
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

