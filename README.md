# Different-CTF-Writeup

<b><h3>Let's start with nmap scanning first</h3></b>

<b>Command:</b> `nmap -vv -sCV -p- 10.10.70.160`

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

<b><h3>We see that http and ftp ports are open. Let's continue by sending a directory scan to the http port</h3></b>
<b>Command:</b> `gobuster dir -u http://10.10.70.160 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 30 2>/dev/null`

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
<h3><b>Let's visit the directory we found</b></h3>

```
Index of /an***********
[ICO]	           Name	                            Last modified       Size	Description
[PARENTDIR]        Parent Directory	 	                        - 	 
[IMG]	           austrailian-bulldog-ant.jpg      2021-01-11 11:51 	58K	 
[TXT]	           wordlist.txt                     2021-01-11 13:48 	394K	 
Apache/2.4.29 (Ubuntu) Server at 10.10.70.160 Port 80
```
<h3><b>Let's get these two files</b></h3>
<b>Command:</b> `wget http://10.10.70.160/announcements/austrailian-bulldog-ant.jpg && wget http://10.10.70.160/announcements/wordlist.txt`

<h3><b>Let's brute the photo using stegcracker</b></h3>
<b>Command:</b> `stegcracker austrailian-bulldog-ant.jpg wordlist.txt`

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
<h3><b>Now we can look at our photo with steghide</b></h3>
<b>Command:</b> `steghide extract -sf austrailian-bulldog-ant.jpg`

```
Enter passphrase: 
wrote extracted data to "user-pass-ftp.txt".
```
<h3><b>Let's read the file named user-pass-ftp.txt</b></h3>
<b>Command:</b> `cat user-pass-ftp.txt`

```
RlRQLUxPR0lOClVT****************************M2FkYW5hY3JhY2s=
```
<h3><b>Encrypted with base64</b></h3>
<b>Command:</b> `cat user-pass-ftp.txt |base64 -d`

```
FTP-LOGIN
USER: h******p
PASS: 1***********k
```
<h3><b>Pretty good ... We can now connect via ftp port</b></h3>
<b>Command:</b> `ftp 10.10.70.160` & `ls -la`

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


