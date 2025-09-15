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

***ADD PIC 1

>We find open ports: 21 - 22 - 80


Question 3 asks? Who wrote the task list?
This gives us a hint that there is a file that can be read
Lets see if we can log on to the ftp server
Logging in with no username or password gives us the output that we are able to login as "anonymous"

***ADD PIC 2 

Lets try anonymous with no password...


***ADD PIC 3

As you can see the ftp server allows login as the anonymous user without authentication
We can see that there are two interesting files

***ADD PIC 4

Using the following commands will allow us to download multiple files to our machine


>prompt
>mget file1 files2 etc...


***ADD PIC 5


Now that we have the two downloaded text files on our machine, 
Lets inspect the task file


***ADD PIC 6


We now have our answer. 
Who wrote the task list?

>Answer: lin


Turning our attention to the "locks.txt" file
We can see what looks to be a list of passwords


***ADD PIC 7

Our next question asks? What service can you bruteforce with the text file found?
Remembering our Nmap scan, lets see if we can bruteforce SSH access with the newly found username and password list

We will use this script with hydra to attempt ssh bruteforce

>:$ hydra -l <user> -P <passwd_file> ssh://<target_ip>

Looking at our scan results we can that hydra was able to find a valid password in the text file

**ADD PIC 8 

This gives us the answer to the next question
What is the users password? 


>Answer: RedDr4gonSynd1cat3

Lets now log in via SSH with our username and password

>username - lin     password - RedDr4gonSynd1cat3

Immediately we are able capture the USER FLAG


***ADD PIC 9 


This give us our answer for the USER FLAG


>Answer: THM{CR1M3_SyNd1C4T3}


Lets attempt to excalate our privilege in order to capture the root flag

>sudo -l 

Running the command above shows that we have sudo permissions for /bin/tar


***ADD PIC 10


Searching [GTFObin](https://gtfobins.github.io/gtfobins/tar/#sudo) for tar shows us that we have the ability to escalate our privilege and commands as root
Lets run the command below


>:$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh


Afer running this command we can now see that we are the root user by running the command

>whoami

Lets now move into the root directory and capture the root flag and answer the final question


***ADD PIC 11


Answer: THM{80UN7Y_h4cK3r}


lESSONS:
Going forward, ensure you have the necessary permissions or context before executing commands. Use sudo for elevated permissions or verify your current location with pwd.
Attempting to access users.txt and /root without the proper permissions or context can lead to unnecessary errors.




