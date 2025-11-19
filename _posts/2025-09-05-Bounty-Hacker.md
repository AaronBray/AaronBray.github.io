---
layout: post
title: Bounty Hacker
date: 2025-09-05 16:25:06
tags: tryhackme ftp
description: TryHackMe Bounty Hacker Write-Up
---
# BOUNTY HACKER 
-TRYHACKME CTF WRITEUP- 
![img]({{ '/assets/images/bountyhacker/bountyhacker.jpeg' | relative_url }}){: .center-image }
[Link To CTF](https://tryhackme.com/room/cowboyhacker)




We start this CTF off with an IP address and a series of challenge questions
Our first two questions tell us to start the target machine and enumerate the open ports

Lets run a basic Nmap scan against the target IP

![img]({{ '/assets/images/bountyhacker/1-bountyhacker.png' | relative_url }}){: .center-image }

We find open ports: 21 - 22 - 80


Question 3 asks? Who wrote the task list?

This gives us a hint that there is a file that can be read
Lets see if we can log on to the ftp server
Logging in with no username or password gives us the output that we are able to login as "anonymous"

![img]({{ '/assets/images/bountyhacker/2-bountyhacker.png' | relative_url }}){: .center-image }

Lets try to log in as anonymous with no password...


![img]({{ '/assets/images/bountyhacker/3-bountyhacker.png' | relative_url }}){: .center-image }

As you can see the ftp server allows login as the anonymous user without authentication

We can see that there are two interesting files

![img]({{ '/assets/images/bountyhacker/4-bountyhacker.png' | relative_url }}){: .center-image }

Using the following commands will allow us to download multiple files to our machine

{% highlight bash %}
$ prompt
$ mget file1 file2 file3 etc...
{% endhighlight bash %}

![img]({{ '/assets/images/bountyhacker/5-bountyhacker.png' | relative_url }}){: .center-image }


Now that we have the two downloaded text files on our machine
Lets inspect the task file


![img]({{ '/assets/images/bountyhacker/6-bountyhacker.png' | relative_url }}){: .center-image }


We now have our answer. 
Who wrote the task list?

>Answer: lin


Turning our attention to the "locks.txt" file
We can see what looks to be a list of passwords


![img]({{ '/assets/images/bountyhacker/7-bountyhacker.png' | relative_url }}){: .center-image }

Our next question asks? What service can you bruteforce with the text file found?

Remembering our Nmap scan, 
Lets see if we can bruteforce SSH access with the newly found username and password list


We will use this script with hydra to attempt ssh bruteforce

{% highlight bash %}
$ hydra -l <user> -P <passwd_file> ssh://<target_ip>
{% endhighlight bash %}

Looking at our scan results we can that hydra was able to find a valid password


![img]({{ '/assets/images/bountyhacker/8-bountyhacker.png' | relative_url }}){: .center-image }



This gives us the answer to the next question

What is the users password? 


>Answer: [redacted]


Lets now log in via SSH with our username and password

{% highlight bash %}
Username: lin     Password: [redacted]
{% endhighlight bash %}


Immediately we are able capture the USER FLAG



![img]({{ '/assets/images/bountyhacker/9-bountyhacker.png' | relative_url }}){: .center-image }


This give us our answer



>Answer: [redacted]




Lets attempt to escalate our privilege in order to capture the root flag

{% highlight bash %}
$ sudo -l 
{% endhighlight bash %}

Running the command above shows that we have sudo permissions for /bin/tar


![img]({{ '/assets/images/bountyhacker/10-bountyhacker.png' | relative_url }}){: .center-image }



Searching [GTFObins](https://gtfobins.github.io/gtfobins/tar/#sudo) for tar 
We can see that we have the ability run a tar command as root
Lets run the command below

{% highlight bash %}
$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
{% endhighlight bash %}

After running the command above, 
We can now see that we are the root user by running the 'whoami' command
{% highlight bash %}
$ whoami
{% endhighlight bash %}

Lets now move into the root directory and capture the root flag to answer the final question


![img]({{ '/assets/images/bountyhacker/11-bountyhacker.png' | relative_url }}){: .center-image }


We now have our ROOT FLAG !



>Answer: [redacted]










# LESSONS LEARNED:
This machine was able to be exploited due to an easily accessible password file
We were able to brute force access into an ssh session
We then escalated our privilege to uncover restricted files

This attack could have been mitigated by ensuring sensitive files are restricted to unauthorized users



