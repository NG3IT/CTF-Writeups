# TryHackMe - Team

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

```bash
Dale
        I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.
```

Let's add this line on this file /etc/hosts :

```bash
team.dev <target_ip>
```


---

## Exploitation



---

## Flag user



---

## Flag root

