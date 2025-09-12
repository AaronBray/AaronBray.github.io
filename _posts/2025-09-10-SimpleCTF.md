---
layout: post
title: Simple CTF
date: 2025-09-10 16:25:06
tags: tryhackme
description: Simple CTF write-up
---

-We start by running a basic Nmap script scan to enumerate the open ports
Nmap shows ports 21(ftp), 80(http), & 2222(openssh)


Using Dirb to enumerate hidden directories, We find: 
/robots.txt   /simple  /simple/admin 
Opening the webpage in our browser shows that:
-CMS made simple is being used to run the website
-Lets search for CMS exploits

I originally tested the script found on exploitdb and within Kali's searchsploit directory for a possible SQL injection.
-ran script with -u http://xxxx argument --crack and -w (tried without --crack) - no success - did not work

Eventually found an alternate exploit script on GitHub for CMS made simple vulnerability
This exploit Worked!
source:  https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3
Running the script against the target IP shows:

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
________________

![img]({{ '/assets/images/deer.jpg' | relative_url }}){: .center-image }Caption test

______________________


We know have an email, username, password and hash.
Lets attempt to bruteforce the hash
check hashes.com and crack station - nothing found
hashes.com Id hash as md5
Tested john the ripper 
Didnt work as expected and could not get any reliable output or any results
Switched to hashcat -Way Easier
Played around with possible -m options for the has after running auto detect mode
Nothing worked
Ended up adding salt to the end of hash  -> passwordhash:salt
0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
ran autodetect mode and got differenct -m options to try
second option was -m 20 (md5($salt.$pass))
WORKED!

***HASH CRACKED!!! --- password is -> secret

logged into webpage /simple/admin/login
logged in as mitch 

logged into ssh session on port 2222
ran "/bin/bash" to run shell normal
found flag in user.txt


***FLAG #1!!!!

looked for ways to get root
find / -type f -perm -04000 -ls 2>/dev/null-- listed files with suid set - nothing promising
getcap -r / 2>/dev/null -- list enabled binary capabilities -- nothing promising
ran sudo -l --FOUND VIM is allowed with sudo 
Search FTGObins and found command:
sudo vim -c ':!/bin/sh'

****GOT ROOT!!!!

searched directories for root flag
found root.txt under /root

GOT ROOT FLAG!!!!!!!!!






Lessons:
if one exploit doesn't work- dont give up- find another ie. GitHub
johntheripper sucks/will take more practice
hashcat is easy
play around with different -m output and adding salt to beginning or end 
