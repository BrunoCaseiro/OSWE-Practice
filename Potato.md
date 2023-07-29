# Potato

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-29 14:23 WEST
Nmap scan report for 10.0.2.37
Host is up (0.00050s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Potato company
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.91 seconds
````

## FTP at port 2112
Anonymous login worked, let's take a look at those files with 'get _filename_' for each

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/67b7bae3-cb69-4745-a3d8-e0b4d1d76242)

welcome.msg

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/0a002d29-258a-4eb2-9952-9e1b86d73bb3)

index.php.bak

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d6eb5193-1380-4a85-92fb-b482dcc83f1b)

Seems like we're about to face a fun code review challenge

## Port 80

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bce23582-2bb0-468b-bc10-3b59dcf3ecb2)


I found the directory /potato, which is empty, and /admin which shows a login form

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/af94caa1-7aa6-4972-b337-a36b1ade8a4d)

This is the likely vulnerable code

````
<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>
````

The first condition just forces us to set the GET parameter 'login' to 1, that's farily easy to do and I know I'm getting past that part since I can get the "Bad login/password" error.

The second condition is only passed if the POST parameter 'username' is set to 'admin' AND the POST parameter 'password' is set to the value of the parameter $pass (which is NOT 'potato', tested)

We need to figure out something for the password parameter that strcmp-ed with a normal string (i think?) results in a 0. From the PHP strcmp documentation

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bf708e16-cc15-4ae5-ab86-99d47fcc6020)

This is the important bit

``strcmp("foo", array()) => NULL + PHP Warning``

So, if we can send an array and compare it to the password, which should be a string, the result is NULL. For some PHP reason, NULL == 0 is true.

Hell yeah!!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/a6f5ecd5-238f-418c-9b2e-7ed63686bdc1)

See that cookie value? That's the password, so we can easily login in the browser

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/fb29a47c-e969-4472-9e8b-54d5e6d8e2b9)


This page in specific is vulnerable to LFI

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/eb2e5b39-701e-4724-a0f8-50ff560d6f38)


![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d2cece81-c444-43d6-8b3d-728beb023512)


Ups there's a hash leak there at the bottom of the file

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d8ebbe6a-9678-41ad-9a75-384f7dfc6659)


And it was cracked immediately

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/fd653969-1267-4a83-9d78-f8a075ea825a)


## Privilege Escalation

These work for SSH

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/99386bcf-f791-484e-a1a0-82addac4fd76)


First flag right here

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/22ea7b04-15b8-497e-a0b5-097e5eb6b0ca)


The home folder of the user florianges is empty, but we have interesting sudo permissions

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/8bc441e5-9ec5-4c92-9313-fcd47d4084de)

Calling clear.sh with nice just clears the terminal. The id script calls id. So I'll assume that /bin/nice simply runs a command.

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/f1de4c96-d8a1-46b2-a5ba-6524f4a96190)

If I can run with sudo any script that's inside /notes/....

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bdffc9a1-c995-43f1-946a-dc9100af7023)

It does fit the wildcard regex :D

Rooted!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/4ef6b083-18bf-4b57-b1f2-9ebec634be94)

