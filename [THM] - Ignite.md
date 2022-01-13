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



---

## User flag



---

## Root flag


