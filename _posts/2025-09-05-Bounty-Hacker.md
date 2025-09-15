---
layout: post
title: Coming Soon !
date: 2025-09-05 16:25:06
tags: tryhackme
description: TryHackMe Bounty Hacker Write-Up
---
# BOUNTY HACKER TRYHACKME WRITEUP

We start this CTF off with an IP address and a series of challenge questions
Our first two questions tell us to start the target machine and enumerate the open ports

Lets run a basic Nmap scan against the target IP

![img]({{ '/assets/images/1-bountyhacker.png' | relative_url }}){: .center-image }

We find open ports: 21 - 22 - 80


Question 3 asks? Who wrote the task list?
This gives us a hint that there is a file that can be read
Lets see if we can log on to the ftp server
Logging in with no username or password gives us the output that we are able to login as "anonymous"

![img]({{ '/assets/images/2-bountyhacker.png' | relative_url }}){: .center-image }

Lets try anonymous with no password...


![img]({{ '/assets/images/3-bountyhacker.png' | relative_url }}){: .center-image }

As you can see the ftp server allows login as the anonymous user without authentication
We can see that there are two interesting files

![img]({{ '/assets/images/4-bountyhacker.png' | relative_url }}){: .center-image }

Using the following commands will allow us to download multiple files to our machine

{% highlight bash %}
>prompt
>mget file1 files2 etc...
{% endhighlight bash %}

![img]({{ '/assets/images/5-bountyhacker.png' | relative_url }}){: .center-image }


Now that we have the two downloaded text files on our machine, 
Lets inspect the task file


![img]({{ '/assets/images/6-bountyhacker.png' | relative_url }}){: .center-image }


We now have our answer. 
Who wrote the task list?

>Answer: lin


Turning our attention to the "locks.txt" file
We can see what looks to be a list of passwords


![img]({{ '/assets/images/7-bountyhacker.png' | relative_url }}){: .center-image }

Our next question asks? What service can you bruteforce with the text file found?
Remembering our Nmap scan, lets see if we can bruteforce SSH access with the newly found username and password list

We will use this script with hydra to attempt ssh bruteforce

{% highlight bash %}
>:$ hydra -l <user> -P <passwd_file> ssh://<target_ip>
{% endhighlight bash %}

Looking at our scan results we can that hydra was able to find a valid password in the text file


![img]({{ '/assets/images/8-bountyhacker.png' | relative_url }}){: .center-image }



This gives us the answer to the next question
What is the users password? 


>Answer: RedDr4gonSynd1cat3

Lets now log in via SSH with our username and password

{% highlight bash %}
>username - lin     password - RedDr4gonSynd1cat3
{% endhighlight bash %}


Immediately we are able capture the USER FLAG


![img]({{ '/assets/images/9-bountyhacker.png' | relative_url }}){: .center-image }


This give us our answer for the USER FLAG


>Answer: THM{CR1M3_SyNd1C4T3}


Lets attempt to excalate our privilege in order to capture the root flag

{% highlight bash %}
:$ sudo -l 
{% endhighlight bash %}

Running the command above shows that we have sudo permissions for /bin/tar


![img]({{ '/assets/images/10-bountyhacker.png' | relative_url }}){: .center-image }


Searching [GTFObin](https://gtfobins.github.io/gtfobins/tar/#sudo) for tar shows us that we have the ability to escalate our privilege and commands as root
Lets run the command below

{% highlight bash %}
:$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
{% endhighlight bash %}

Afer running this command we can now see that we are the root user by running the command

{% highlight bash %}
:$ whoami
{% endhighlight bash %}

Lets now move into the root directory and capture the root flag and answer the final question


![img]({{ '/assets/images/11-bountyhacker.png' | relative_url }}){: .center-image }


We now have our ROOT FLAG!

>Answer: THM{80UN7Y_h4cK3r}


lESSONS:
Going forward, ensure you have the necessary permissions or context before executing commands. Use sudo for elevated permissions or verify your current location with pwd.
Attempting to access users.txt and /root without the proper permissions or context can lead to unnecessary errors.




