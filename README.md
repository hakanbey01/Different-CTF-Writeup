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
[ICO]	           Name	                                 Last modified	           Size	Description
[PARENTDIR]        Parent Directory	 	           - 	 
[IMG]	           austrailian-bulldog-ant.jpg      2021-01-11 11:51 	58K	 
[TXT]	           wordlist.txt                     2021-01-11 13:48 	394K	 
Apache/2.4.29 (Ubuntu) Server at 10.10.70.160 Port 80
```
aaaa
