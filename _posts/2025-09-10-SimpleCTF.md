---
layout: post
title: Simple CTF
date: 2025-09-10 16:25:06
tags: tryhackme
description: Simple CTF write-up
---

-We start by running a basic Nmap script scan to enumerate the open ports

{% highlight bash %}
>:$ Nmap -sC <target_ip>
{% endhighlight bash %}


Nmap shows ports: 21(ftp) -- 80(http) -- & 2222(openssh)

Turning our attention to the web server on port 80, we can use Dirb or gobuster to enumerate hidden directories. 
{% highlight bash %}
dirb http://<target_ip> 
{% endhighlight bash %}
>This will run a basic directory scan using the common.txt file located in /usr/share/wordlists/dirb/commmon.txt

We find:
{% highlight bash %}
-/robots.txt   
-/simple  
-/simple/admin 
-/simple/admin/login
{% endhighlight bash %}

Opening the webpage in our browser shows that:
-CMS made simple is being used to run the website
-Lets search for CMS exploits

I tested and played around the CMS made simple exploit found on exploitdb as well as within Kali's searchsploit (exploit) directory for a possible SQL injection.

Eventually found an alternate exploit script on GitHub for CMS made simple vulnerability
This exploit Worked!
source:  [github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3](https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3)

Running this exploit script against the target IP shows:
{% highlight bash %}
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
{% endhighlight bash %}
________________

![img]({{ '/assets/images/deer.jpg' | relative_url }}){: .center-image }Caption test

______________________


We now have an email, username, password and hash.
Lets attempt to bruteforce the hash

Checking [hashes.com](https://hashes.com/en/decrypt/hash) and [crack station](https://crackstation.net/) - nothing found
Using hashes.com to identify the hash type shows this as an md5 hash
Moving beyond online hash crackers lets load the hash into a text file and use Hashcate or JohnTheRipper to check against the rockyou password list

Tested john the ripper - Didnt work as expected on windows and could not get any reliable output or any results

Running the hash in hashcat in autodetect mode shows that all -m hashtype options do not work
Lets add the salt to the hash
Adding the salt to the end of hash like so -> passwordhash:salt

-0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2

running autodetect mode now shows differenct -m options to try
Our second option shows that we can use:

-m 20 (md5($salt.$pass))

This hash type works and Hashcat was able to crack the hash!

-Password is -> secret

We know have the username email, and cracked password
Let's login to the webpage at /simple/admin/login
logging in as mitch shows that the password is accepted

Lets now try this login information to access the server via ssh
Remember that ssh is being run on port 2222 (not port 22)

Logging into ssh session on port 2222
We can see that the credentials are accepted
Lets run - "/bin/bash" - to convert into a bash shell to make things easier
Immediately we are shown the user.txt file

Lets cat out this file to uncover the user flag
We know have the initial User flag

+USER FLAG!!!!!!!!!!!!!
_______________________________

We now need to escalate our privilege in order to gain to root flag
Lets look for ways to get root
Searching:
>:$ find / -type f -perm -04000 -ls 2>/dev/null-- listed files with suid set -- nothing promising was found
>:$ getcap -r / 2>/dev/null -- list enabled binary capabilities -- nothing promising was found
>:$ sudo -l -- shows that VIM commands are allowed to be run as sudo 

Searching [FTGObins](https://gtfobins.github.io/) shows that we can use VIM to escalate our privilege:

Running the commnd:
>:$ sudo vim -c ':!/bin/sh'

We can see this command was accepted and we are now the root user
Lets move into the root directory and capture the root flag

We now have the root.txt flag file shown in the /root directory accessible
When we cat out this file we can see that we have now uncovered the root flag!

+ROOT FLAG!!!!!!!!!



Lessons:
try sudo -l first forpriv esc
if one exploit doesn't work- dont give up- find another ie. GitHub, exploitdb, writing your own, etc
johntheripper sucks on windows
hashcat is easier on windows if you know -m hash type
play around with different -m types, and adding salt to beginning or end of hash
