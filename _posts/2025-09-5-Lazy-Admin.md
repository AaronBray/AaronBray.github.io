---
layout: post
title: Coming Soon!
date:   2025-09-5 11:07
description: tryhackme Lazy Admin CTF write-up
tags: tryhackme
comments: false
---

#LAZY ADMIN WRITEUP 

-Found port 22/80 open with nmap
-enumerated directories with dirb
-/content shows site is running CMS sweetRice
-found login page at /content/as -
- tried basic sql injection -- failed
-tried sweetrice online file upload vulnerability
-found directory list and site map at /content/inc
-found sql database backup file
-found entry for admin with hashed password

admin
manager
42f749ade7f9e195bf475f37a44cafcb = Password123

-cracked hash with online crack tool
-tried admin -- failed-
-tried manager - WORKED!!!

-tried to upload reverse php shell - failed - no shell
-created new reverse shell - fatrat- php - laptop died - hung out with sis and didn't try
-used pentestmonkey reverse php shell - uploaded to media form
-used burpsuite to modify to request to bypyass filter and sent form and .phtml instead of .php
-GOT USER FLAG

(-netcat super unstable dropped connection 2-3 times)

-checked walkthough to get hint to next step
-sudo -l shows that there is a 'backup.pl" file which shows /etc/copy.sh
-/etc/copy is a reverse shell script to complete a connection for a remote listener
-modified script to point to my ip
-ran command "sudo perl /home/itguy/backup.pl
-used alternate port to complete connection
-received root reverse shell

catted out /root/root.txt to get ROOT FLAG
