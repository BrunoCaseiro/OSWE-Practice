# Pipe

## Nmap
````
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-30 19:58 WEST
Nmap scan report for 10.0.2.39
Host is up (0.00043s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 16:48:50:89:e7:c9:1f:90:ff:15:d8:3e:ce:ea:53:8f (DSA)
|   2048 ca:f9:85:be:d7:36:47:51:4f:e6:27:84:72:eb:e8:18 (RSA)
|   256 d8:47:a0:87:84:b2:eb:f5:be:fc:1c:f1:c9:7f:e3:52 (ECDSA)
|_  256 7b:00:f7:dc:31:24:18:cf:e4:0a:ec:7a:32:d9:f6:a2 (ED25519)
80/tcp  open  http    Apache httpd
|_http-server-header: Apache
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=index.php
|_http-title: 401 Unauthorized
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33320/tcp6  status
|   100024  1          51317/tcp   status
|   100024  1          53835/udp6  status
|_  100024  1          55588/udp   status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.85 seconds
````

## Port 80

Basic auth...
![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bf95838c-cec1-4306-b668-d7118deb204d)


Filebuster found the directory /image which had directory listing enabled and one file...

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/959aeb2f-5759-4e16-a49b-778f8a0fc093)

Some background from wikipedia

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/c17fa88f-a08c-4b05-9d2d-e59cc965ac46)

Strings and exiftool don't show anything interesting

I honestly didn't think this would work, but calling /index.php with POST or DEBUG results in a 200 OK!

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/e7deb98a-b956-4445-9463-bfc2308d1631)


This is the result

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/9d8d2298-982c-4f47-9965-c52052ea834c)

Pressing the artist info link...

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/b6293971-3012-45e2-92c2-915da6477c35)

That parameter is a PHP object. Insecure deserialization?

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/b1016b79-9af2-42bc-b628-75a4b059feee)


![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/0be81a58-3bfa-4d9b-856b-a67145d671ee)


Another clue! Let's look into the scriptz directory

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/d5f85df6-1212-4138-b941-43cf4266379b)


![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/b9fb9a67-e9f0-493f-9229-c419a11ae033)

The log file...

````
<?php
class Log
{
    public $filename = '';
    public $data = '';

    public function __construct()
    {
        $this->filename = '';
	$this->data = '';
    }

    public function PrintLog()
    {
        $pre = "[LOG]";
	$now = date('Y-m-d H:i:s');

        $str = '$pre - $now - $this->data';
        eval("\$str = \"$str\";");
        echo $str;
    }

    public function __destruct()
    {
	file_put_contents($this->filename, $this->data, FILE_APPEND);
    }
}
?>
````

This file is being included, the goal here is to call an insecure deserialization with this class

I sent this url encoded in the 'param' POST parameter

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/23ab4bb8-d28f-419a-993c-ba73e2880690)

And tested it out... again, DEBUG method

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/cb72d91b-1b9a-48e6-97f1-445a630373a3)

This.

Is.

Awesome.


## Reverse Shell

Now to repeat the same thing but with a more dangerous payload

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/15411059-4054-48ae-a9bc-dd2895a4fbac)

I had to fight a lot with this, apparently it only worked inside the scriptz directory for some reason

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/7977dee8-2b5f-4f08-8718-b9c0279eaf6d)

Sending a POST request with the cmd parameter to 'id'

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/9c39c53d-78ee-4ee2-8bb9-e77ebf505265)

Time for mkfifo... and I'm in

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/b37ab874-8f53-437a-ab41-5cefdeaf7603)


## Privilege Escalation

We have these cronjobs

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/532bc963-70d6-497f-81ba-13b7b5f6e829)

compress.sh

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/259d6b8d-2e21-476d-a0eb-7708b078798b)


The old tar wildcard!

First you create these two files...

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/bb079b84-e0a0-4d8d-aec7-c0b98b16333e)


And then the actual payload goes in a shell.sh file

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/33694e57-8a1c-438c-8440-fa478ae13e7c)

We wait a few minutes and root

![image](https://github.com/BrunoCaseiro/OSWE-Practice/assets/38294180/926186b9-1111-4759-b938-d3857f211095)

