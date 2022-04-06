# HTB_Paper
Documenting my first attempt at the Paper box on Hack the Box.

First I visit the IP given to me at 10.129.112.255 and I see a website:
![image](https://user-images.githubusercontent.com/96757616/162024517-753b3824-cb91-4e18-aebb-bf14bb72c71e.png)

I see no login screens or anything I can apparently interact with at this time.

Step 1:
Nmap scan as follows:
nmap -sC -sV 10.129.112.255

RESULTS:

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-06 17:39 BST
Nmap scan report for 10.129.112.255
Host is up (0.076s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
80/tcp  open  http     Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: HTTP Server Test Page powered by CentOS
443/tcp open  ssl/http Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: HTTP Server Test Page powered by CentOS

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.50 seconds

I see open ports 80, 443 (HTTP / apache server) and 22 (TCP/SSL).

cURL -I 10.129.112.255:

HTTP/1.1 403 Forbidden
Date: Wed, 06 Apr 2022 17:20:07 GMT
Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
X-Backend-Server: office.paper
Last-Modified: Sun, 27 Jun 2021 23:47:13 GMT
ETag: "30c0b-5c5c7fdeec240"
Accept-Ranges: bytes
Content-Length: 199691
Content-Type: text/html; charset=UTF-8

I see "X-BACKEND-SERVER: OFFICE.PAPER" and decide to add that to the etc/hosts file so I can view it's contents

Lo and behold we get a Dunder Mifflin ripoff! Blunder Tifflin has some blog posts, one of them reads that one of his draft posts isn't as hidden as he thinks. Using Wappalyzer I see that there are several exploits possible for this version of wordpress.

Exploit for WordPress which allows unauthenticated users to view sensitive information using a ?static=1 flag in the URL is present using WPSCAN tool leads me to try this and I find an internal company blog post revealing that there is sensitive information being stored on "drafts":

----------------------------------------------------------------------------------------------------------------------------------------------------------

test

Micheal please remove the secret from drafts for gods sake!

Hello employees of Blunder Tiffin,

Due to the orders from higher officials, every employee who were added to this blog is removed and they are migrated to our new chat system.

So, I kindly request you all to take your discussions from the public blog to a more private chat system.

-Nick

# Warning for Michael

Michael, you have to stop putting secrets in the drafts. It is a huge security issue and you have to stop doing it. -Nick

Threat Level Midnight

A MOTION PICTURE SCREENPLAY,
WRITTEN AND DIRECTED BY
MICHAEL SCOTT

[INT:DAY]

Inside the FBI, Agent Michael Scarn sits with his feet up on his desk. His robotic butler Dwigt….

# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY

# I am keeping this draft unpublished, as unpublished drafts cannot be accessed by outsiders. I am not that ignorant, Nick.

# Also, stop looking at my drafts. Jeez!
Recent Posts

    Feeling Alone!
    Secret of my success
    Hello Scranton!

Recent Comments

    nick on Feeling Alone!
    Creed Bratton on Hello Scranton!
-------------------------------------------------------------------------------------------------------------------------------------------------------------

Spotted a link to a chat - http://chat.office.paper/register/8qozr226AhkCHZdyY 

When I paste this link, I get an error:
![image](https://user-images.githubusercontent.com/96757616/162052101-74fbfe7f-545d-455c-a17f-418301659ae6.png)

so perhaps add chat.office.paper to etc/hosts as well?

added the chat.office.paper to etc/hosts section by appending right after the previous addition and bingo I'm in and am presented with a chat new user registration form.

used Linux-type commands via private message to the bot to explore the file directories to locate /hubot/env via ../list commands and ultimately after some googling found that I could access these files with file../hubot/.env which gave me a password list

![image](https://user-images.githubusercontent.com/96757616/162063771-7c4eee07-8d7d-43b9-bfeb-1709541b9737.png)

after trying to login I get a warning that bots can't login via this web console, so I checked to see what other areas I can use these credentials on and remembered the SSH I uncovered during nmap scan:

SSH into recyclops@10.129.112.255 with password Queenofblad3s!23 with no luck
SSH into dwight@10.129.112.255 with password Queenofblad3s!23 and bingo!

checked to see that we have a file which appears to be a flag named user.txt

![image](https://user-images.githubusercontent.com/96757616/162065421-0da75c5f-91fe-4ade-a43e-63ed4a2f0a3d.png)


