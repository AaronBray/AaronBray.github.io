---
layout: post
title: Simple CTF
date: 2025-09-01 16:25:06
tags: tryhackme sqli VIM 
description: TryHackMe SimpleCTF CTF Write-Up 
---

# SIMPLE CTF 
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/easyctf)

Let's start by running a Nmap scan to enumerate all open ports with the -p- argument

{% highlight bash %}
$ Nmap -p- <target_ip>
{% endhighlight bash %}


Nmap shows open ports: 21, 80, & 2222


![img]({{ '/assets/images/simplectf/1-simple-ctf.png' | relative_url }}){: .center-image }

Question #1 asks? How many services are running under port 1000?


>Answer: 2


Question #2 asks? What is running on the higher port?
Adding the -sV flag and specifying port 2222 will give us our second answer.

![img]({{ '/assets/images/simplectf/2-simple-ctf.png' | relative_url }}){: .center-image }




>Answer: ssh



Turning our attention to the web server on port 80, we can use Dirb or gobuster to enumerate hidden directories. 
{% highlight bash %}
dirb http://<target_ip> 
gobuster dir -u http://<target_ip> -w /path/to/wordlist/ 
{% endhighlight bash %}


Using Dirb we find:

{% highlight bash %}
/robots.txt   
/simple  
/simple/admin 
/simple/admin/login
{% endhighlight bash %}



Double checking our results with gobuster we show: /simple


![img]({{ '/assets/images/simplectf/4-simple-ctf.png' | relative_url }}){: .center-image }

Enumerating this directory further shows us the /simple/admin directory

![img]({{ '/assets/images/simplectf/5-simple-ctf.png' | relative_url }}){: .center-image }

Opening /simple in our web browser shows "This site is powered by CMS Made Simple version 2.2.8"
Navigating to /simple/admin redirects us to /simple/admin/login.php and shows a login page also displaying CMS Made Simple

![img]({{ '/assets/images/simplectf/6-simple-ctf.png' | relative_url }}){: .center-image }

A quick google search shows:
"CMS Made Simple is a free, open-source content management system (CMS) that provides a web-based interface for developers and site owners to manage websites. It is written in PHP and is known for its flexibility and ease of use..."

Lets search for CMS Made Simple exploits...

I originally tested and played around the CMS made simple exploit found on exploitdb as well as within Kali's searchsploit (exploit) directory for a possible SQL injection.
I wasnt able to get any of these exploits to work unfortunately. I decided to check online again for another exploit.

Eventually found an alternate exploit script on GitHub for a CMS made simple sqli vulnerability
Source:  [github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3](https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3)


This exploit Worked as well as gave us our answers to questions 3 and 4
What's the CVE you're using against the application? 
To what kind of vulnerability is the application vulnerable?

>Answer: CVE-2019-9053
>Answer: sqli


Running this exploit script against the target IP shows:


>$ python3 exploit.py -u http://<target_ip>/simple/ --crack -w /usr/share/wordlists/rockyou.txt


![img]({{ '/assets/images/simplectf/8-simple-ctf.png' | relative_url }}){: .center-image }

______________________


We now have an email, username, and unencrypted password
This gives us our 5th answer

Question 5 asks? What's the password?


>Password is -> secret


__________________________
Just for fun, Lets attempt to crack the hash for extra practice

Checking [hashes.com](https://hashes.com/en/decrypt/hash) and [crack station](https://crackstation.net/) - Both sites failed to crack the hash
Lets load the hash into a text file and use Hashcate or JohnTheRipper to check against the rockyou password list

Using hashes.com to identify the hash type shows this as an md5 hash

Testing the hash without the salt in hashcat using autodetect mode resuled in no success
Lets add the salt to the hash and see if that works
Adding the hash to the beggining is outputting errors 
We will need to add the salt to the end of hash -> passwordhash:salt


>0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2


Running autodetect mode now shows different -m options to try


![img]({{ '/assets/images/simplectf/9-simple-ctf.png' | relative_url }}){: .center-image }


Our second option shows that we can use:

>-m 20 (md5($salt.$pass))



Using the -m option, -a 0 option for wordlist attack mode, and the rockyou.txt password file 
We are able to crack the hash!


>hashcat.exe -m 20 -a 0 hash.txt passwd_list.txt

![img]({{ '/assets/images/simplectf/10-simple-ctf.png' | relative_url }}){: .center-image }


__________________________________________________________


We now have the username email, and cracked password
Let's login to the webpage at /simple/admin/login.php
Logging in as mitch shows that the password is accepted!

Lets see where else we can use this information to login
Lets try this login information to access the server via ssh
Remember that ssh is being run on port 2222 (not port 22)

Logging into an ssh session as mitch on port 2222
We can see that the credentials are accepted
{% highlight bash %}
$ ssh mitch@<ip> -p 2222
{% endhighlight bash %}


This gives us our sixth answser
Question 6 asks? Where can you login with the details obtained?

>Answer: ssh

Lets run:
{% highlight bash %}
"/bin/bash"
{% endhighlight bash %}
This will convert us into a bash shell
Immediately we are shown the user.txt file


![img]({{ '/assets/images/simplectf/11-simple-ctf.png' | relative_url }}){: .center-image }


Lets open this file to uncover the user flag
We now have the initial user flag and our 7th answer


>Answer: USER FLAG [not shows on purpose]


______________________________________________________________
Question 6 asks? Is there any other user in the home directory? What's its name?
Looking around the directories shows us another user 

![img]({{ '/assets/images/simplectf/12-simple-ctf.png' | relative_url }}){: .center-image }

>Answer: sunbath

We now need to escalate our privilege in order to gain to root flag
Lets look for ways to get root
Running:
{% highlight bash %}
$ sudo -l
{% endhighlight bash %}
We see that VIM commands are allowed to be run as sudo

![img]({{ '/assets/images/simplectf/13-simple-ctf.png' | relative_url }}){: .center-image }

Searching [FTGObins](https://gtfobins.github.io/) shows that we can use VIM to escalate our privilege:

Running the commnd:
{% highlight bash %}
$ sudo vim -c ':!/bin/sh'
{% endhighlight bash %}
We can see this command was accepted and we are now the root user

This gives our our seventh answer
>Answer: VIM

Lets move into the root directory and capture the root flag

The root.txt flag file shown in the /root directory should now be accessible
When we cat out this file we can see that we have now captured the root flag!


![img]({{ '/assets/images/simplectf/14-simple-ctf.png' | relative_url }}){: .center-image }

>Answer: ROOT FLAG!!!!!!!!! [not shown on purpose]



# LESSONS LEARNED:
This machine was able to be exploited through a web app that was vulnerable to a known sqli attack.
I was able to enumerate username, email, and password due to a lack of input validation/sterlization.
Exploiting a vulnerability using a sudo VIM privilege escalation script allowed us to gain root access and uncover retricted files. 

- Updating the webapp, Enforcing stricter password requirements, as well as increasing validation/sterlization efforts would prevent this type of attack in the future.
