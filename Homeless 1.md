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


#### Reverse Shell




## Privilege Escalation
