# [THM] - Mr Robot CTF

[Room link](https://tryhackme.com/room/mrrobot)

---

## Enumeration

```bash 
# Quick full nmap
$ nmap -sT -T4 -p- -oN full_nmap.txt 10.10.226.108
PORT    STATE  SERVICE 
22/tcp  closed ssh  
80/tcp  open   http
443/tcp open   https

# Detail nmap
$ nmap -A -p22,80,443 -oN details_nmap.txt 10.10.226.108
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open   ssl/http Apache httpd
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
```   

### HTTP 80

Nothing seems really interesting with the interactive prompt simulated on the website. 

```bash
# Gobuster : discovery about website content
$ gobuster -u http://10.10.226.108 -w /usr/share/seclists/25/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,bak

/index.html (Status: 200)                                                                                                                                                                                          
/index.php (Status: 301)                                                                                                                                                                                           
/images (Status: 301)                                                                                                                                                                                              
/blog (Status: 301)                                                                                                                                                                                                
/rss (Status: 301)                                                                                                                                                                                                 
/sitemap (Status: 200)                                                                                                                                                                                             
/login (Status: 302)                                                                                                                                                                                               
/0 (Status: 301)                                                                                                                                                                                                   
/feed (Status: 301)                                                                                                                                                                                                
/video (Status: 301)                                                                                                                                                                                               
/image (Status: 301)                                                                                                                                                                                               
/atom (Status: 301)                                                                                                                                                                                                
/wp-content (Status: 301)                                                                                                                                                                                          
/admin (Status: 301)                                                                                                                                                                                               
/audio (Status: 301)                                                                                                                                                                                               
/intro (Status: 200)                                                                                                                                                                                               
/wp-login (Status: 200)
/wp-login.php (Status: 200)
/css (Status: 301)
/rss2 (Status: 301)
/license (Status: 200)
/license.txt (Status: 200)
/wp-includes (Status: 301)
/js (Status: 301)
/wp-register.php (Status: 301)
/Image (Status: 301)
/wp-rss2.php (Status: 301)
/rdf (Status: 301)
/page1 (Status: 301)
/readme (Status: 200)
/readme.html (Status: 200)
/robots (Status: 200)
/robots.txt (Status: 200)
``` 

http://10.10.226.108/wp-login.php -> Authentication form to Wordpress admin panel
http://10.10.226.108/0/ -> Wordpress blog called "User's blod" -> There is no data directly on this blog -> WordPress 4.3.1
http://10.10.226.108/robots -> Good informations here. 
```txt
User-agent: *
fsocity.dic
key-1-of-3.txt
```

```bash
# This is a dictionnary with 858160 patterns
$ cat fsocity.dic | tail -10
ER28-0652
psychedelic
iamalearn
uHack
imhack
abcdefghijklmno
abcdEfghijklmnop
abcdefghijklmnopq
c3fcd3d76192e4007dfb496cca67e13b
ABCDEFGHIJKLMNOPQRSTUVWXYZ
$ cat fsocity.dic | wc -l
858160
```

http://10.10.226.108/fsocity.dic -> 
http://10.10.226.108/key-1-of-3.txt -> 073403c8a58a1f80d943455fb30724b9 -> redirect to user's blog

---

### HTTPS 443

Like website on port 80, nothing seems really interectiong. On first look, this is the same content.

---

## Exploitation



---

## Flag 1



---

## Flag 2



---

## Flag 3


