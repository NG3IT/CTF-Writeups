# TryHackMe - Nax

[Room link](https://tryhackme.com/room/nax)

---

## Enumeration

Nmap's result :

```bash
$ sudo nmap -sS -oN nmap.txt 10.10.245.105                                       1 ⨯
[...]
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
389/tcp open  ldap
443/tcp open  https
```

Full Nmap's result :

```bash
sudo nmap -sS -p- -oN nmap-full.txt 10.10.245.105
[...]
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
389/tcp  open  ldap
443/tcp  open  https
5667/tcp open  unknown
```

<br>

Gobuster on port 80 :

```
$ gobuster dir -u http://10.10.245.105 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt 
[...]
/javascript           (Status: 301) [Size: 319] [--> http://10.10.245.105/javascript/]
/nagios               (Status: 401) [Size: 460]                                       
/server-status        (Status: 403) [Size: 278] 
```

Exploaration on port 80 :

On /

```txt
Welcome to elements.					Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr
```

We can find this elements on the periodic table of elements and then we can retrieve the number associated.

We can use cyberchief for find the secret file. Use "From decimal" option.

```txt
Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr
47 – 80 – 73 – 51 – 84 – 46 – 80 – 78 – 103
```

We can find the artist of this picture with exiftool

```bash
$ exiftool PI3T.PNg
[...]
Artist                          : Piet Mondrian
```

<br>

---

## Exploitation



<br>

---

## User flag



<br>

---

## Root flag



<br>
