a# Secure Code 1

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-29 19:47 WEST
Nmap scan report for 10.0.2.38
Host is up (0.0017s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/login/*
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Coming Soon 2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.44 seconds
````

## Port 80

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/459d18e5-e73f-4c61-960b-0970c69c9730)

The homepage is a simple static page, but the /login endpoint has... a login form :)

GoBuster found the /include/ directory, which seems like it has some interesting files, but I can't open any of them. Maybe futue LFI?

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/e136aa6c-b53e-4471-8b96-91e4219f90f2)


Using the reset password form allows us to enumerate users. Found:

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/c7b31c20-abc8-44b0-824b-3c52aaed84d8)

Not found:

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/acab704b-d5ad-4776-863f-ceab0ca83313)

So I have the username "admin". But still I'm at a loss... 

I needed a hint for this one, and I guess I was missing the file source_code.zip at the root

## Code review time!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/c311140b-da21-465a-9039-b0ca7bce8c37)


Time to read some code. After analyzing every file, there was one that was still under construction. It doesn't include the isAuthenticated.php file, so let's try accessing items in an IDOR kind of way

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/7c72828b-44a4-4241-8d36-ce51a6b0e2a1)

Some are not found

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/6cf72973-283a-478e-942d-c8d68af83c53)


Others are found

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/3f2cacbe-e86c-49d2-a2b0-1c2d8e2eb7be)


It doesn't seem like it's protected against SQLi, so I ran sqlmap and got a hit

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/34a15de2-16ab-4a08-975c-b4d0fb35b581)

Let's enumerate the table and dump the interesting bits (i cut a lot of screenshots here)

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/69ed6338-bd29-47ca-ac7e-a30297a13de6)


The passwords are not accessible, but the tokens are. Let's see how we can change the admin's password

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/0deefe81-5643-4bbc-8e28-330683544bf5)


Okay, so let's add the token we dumped from the db and change the password to 'admin'

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a1d9d193-d45e-4812-ac07-ece1cc42db28)



## Reverse Shell

I'm finally logged in as admin!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/9a2f9c46-778e-42c3-9e99-b45f0b3e92f4)

In the "Items" tab there's a form to upload files

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/912c7d03-92fb-435c-9153-4f2556f20101)


Hmm hard to bypass, but we have the source for this

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d5db092b-4b60-4e94-85cd-c4123393c92f)


Finally got a shell with the .phar extension. Yes I mistyped shell

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/539eb0f6-29c5-4f84-8eed-364a50e87c77)


mkfifo worked for a reverse shell

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/0962534f-4b4c-4e9e-b436-8f3f073828bb)


And that's the end of the box, no root for this.
I might revisit this and write full one click exploits in python. Seems like good practice for the OSWE
