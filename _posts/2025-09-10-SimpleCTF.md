---
layout: post
title: Simple CTF
date: 2025-09-10 16:25:06
tags: tryhackme
description: Simple CTF write-up
---

-We start by running a Nmap scan to enumerate all open ports with the -p- argument

{% highlight bash %}
>:$ Nmap -p- <target_ip>
{% endhighlight bash %}


Nmap shows open ports: 21, 80, & 2222

*Add pic 1-nmap

Question #1 asks? How many services are running under port 1000?

>Answer: 2

Question #2 asks? What is running on the higher port?
Adding the -sV flag and specifying port 2222 will give us our second answer.

*Add pic 2-nmap

>Answer: ssh

Turning our attention to the web server on port 80, we can use Dirb or gobuster to enumerate hidden directories. 
{% highlight bash %}
dirb http://<target_ip> --This will run a basic directory scan using the common.txt file located in /usr/share/wordlists/dirb/commmon.txt
gobuster dir -u http://<target_ip> -w /path/to/wordlist/ -r  -- (-r) instructs gobuster to follow HTTP redirects
{% endhighlight bash %}


Using Dirb we find:
{% highlight bash %}
-/robots.txt   
-/simple  
-/simple/admin 
-/simple/admin/login
{% endhighlight bash %}

*Add dirb pic

Double checking our results with gobuster we show the /simple directory


*Add gobuster pic 1

Enumerating this directory further shows us the /simple/admin page

*Add gobuster pic 2

Opening /simple in our web browser shows "This site is powered by CMS Made Simple version 2.2.8"
Navigating to /simple/admin redirects us to /simple/admin/login.php and shows a login page also displaying CMS Made Simple
A quick google search shows:
"CMS Made Simple is a free, open-source content management system (CMS) that provides a web-based interface for developers and site owners to manage websites. It is written in PHP and is known for its flexibility and ease of use..."

Lets search for CMS Made Simple exploits...

I originally tested and played around the CMS made simple exploit found on exploitdb as well as within Kali's searchsploit (exploit) directory for a possible SQL injection.
I wasnt able to get any of these exploits to work unfortunately. I decided to check online again for another exploit.
Eventually found an alternate exploit script on GitHub for a CMS made simple sqli vulnerability
Source:  [github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3](https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3)
This exploit Worked as well as gave us our third and fourth answer

>Answer: CVE-2019-9053
>Answer: sqli


Running this exploit script against the target IP shows:
>python3 46635.py -u http://<target_ip>/simple/ --crack -w /usr/share/wordlists/rockyou.txt
{% highlight bash %}
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
{% endhighlight bash %}
________________

*add python pic

______________________


We now have an email, username, password and hash.
Lets attempt to bruteforce the hash

Checking [hashes.com](https://hashes.com/en/decrypt/hash) and [crack station](https://crackstation.net/) - Both sites failed to crack the hash
Moving beyond online hash crackers lets load the hash into a text file and use Hashcate or JohnTheRipper to check against the rockyou password list
Using hashes.com to identify the hash type shows this as an md5 hash

Testing the hash without the salt in hashcat using autodetect mode shows that all the -m hashtype options do not work
Lets add the salt to the hash and see if that works
Adding the hash to the beggining is outputting errors
We will need to add the salt to the end of hash -> passwordhash:salt

>0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2

Running autodetect mode now shows differenct -m options to try
Our second option shows that we can use:

>m 20 (md5($salt.$pass))

using the -m option,-a 0 option for wordlist attack mode, and the rockyou.txt password file we are able to crack the hash!
>hashcat.exe -m 20 -a 0 hash.txt password_list.txt

*Add hashcatpassword

We now get our fifth answer

>Password is -> secret

We now have the username email, and cracked password
Let's login to the webpage at /simple/admin/login
Logging in as mitch shows that the password is accepted!

Lets see where else we can use this information to login
Lets try this login information to access the server via ssh
Remember that ssh is being run on port 2222 (not port 22)

Logging into an ssh session as mitch on port 2222
>ssh mitch@i<p> -p 2222

We can see that the credentials are accepted
This gives us our sixth answser

>Answer: ssh

Lets run - "/bin/bash" - to convert into a bash shell to make things easier
Immediately we are shown the user.txt file


*Add user flag pic


Lets open this file to uncover the user flag
We now have the initial user flag and our 7th answer


>Answer: USER FLAG [not shows on purpose]


______________________________________________________________

Looking around the directories shows us another user 

*sunbath pic

>Answer: sunbath

We now need to escalate our privilege in order to gain to root flag
Lets look for ways to get root
Searching:
>:$ sudo -l
We see that VIM commands are allowed to be run as sudo

*sudo pic

Searching [FTGObins](https://gtfobins.github.io/) shows that we can use VIM to escalate our privilege:

Running the commnd:
>:$ sudo vim -c ':!/bin/sh'

We can see this command was accepted and we are now the root user

This gives our our seventh answer
>Answer: VIM

Lets move into the root directory and capture the root flag

We now have the root.txt flag file shown in the /root directory accessible
When we cat out this file we can see that we have now uncovered the root flag!


*Add root pic

>Answer: ROOT FLAG!!!!!!!!! [not shown on purpose]


Lessons:

-If one exploit doesn't work- dont give up- find another ie. GitHub, exploitdb, writing your own, etc
-Hashcat is easier on windows if you know -m hash type and you gain advantage of using gpu to crack password as opposed ti using linux withing a VM
-JohnTheRipper is easier to run on linux than on windows
-Play around with different -m types, as well as adding salt to beginning or end of hash
