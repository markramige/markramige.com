+++
title = "TryHackMe Advent of Cyber 2 Write-Up"
date = "2020-12-25"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["", ""]
keywords = [""]
description = "Intro"
showFullContent = false
+++

Intro

## Contents
* [Day 1 - A Christmas Crisis](#day-1---a-christmas-crisis)
* [Day 2 - The Elf Strikes Back!](#day-2---the-elf-strikes-back)
* [Day 3 - Christmas Chaos](#day-3---christmas-chaos)
* [Day 4 - Santa's Watching](#day-4---santas-watching)

## Day 1 - A Christmas Crisis
### Task 6
* What is the name of the cookie used for authentication?

Open Firefox devtools and go to the Network tab. There is only one cookie for this page.

* In what format is the value of this cookie encoded?

If you don't immediately recognize the format, paste it into [CyberChef](https://gchq.github.io/CyberChef/) and try decoding from different formats until the output makes sense.

* Having decoded the cookie, what format is the data stored in?

This will be obvious once the cookie is decoded.

* What is the value of Santa's cookie?

Replace the username of the cookie from your account and reencode it in the same format using CyberChef.

* What is the flag you're given when the line is fully active?

Replace your cookie with Santa's cookie and reload the page. You should now be able to toggle all of the switches on and the flag will be shown.

## Day 2 - The Elf Strikes Back!
### Task 7
* What string of text needs adding to the URL to get access to the upload page?

Based on the question we know that we need a GET request to gain access to the page. The name of the request is given on the webpage and the value of the request is given on the sticky note in the task text. The format of the request is `?name=value`.

* What type of file is accepted by the site?

We can check in the page source to see what kind of files are accepted in the `<input>` tag. We can also just click on the file select button and the browser UI will show us what types of files are accepted. This doesn't necessarily mean that the server accepts the same types of files, or that there are any checks on the server-side at all.

* In which directory are the uploaded files stored?

Once we've uploaded a file successfully, we can enumerate directories and see if we can guess the correct one. If it turns out not to be easy to guess, then we could use a directory brute-forcing tool such as `ffuf`.

* Activate your reverse shell and catch it in a netcat listener!

Assuming we've configured the `php-remote-shell.php` file correctly and run our listening, then it's just a matter of navigating to the remote shell script in the browser and it will automatically connect to our listener.

* What is the flag in /var/www/flag.txt?

Since our remote shell is now connected, we can just do `cat /var/www/flag.txt` to get the flag!

## Day 3 - Christmas Chaos
### Task 8
* What is the flag?

This task is a good demonstration of the power of Burp Suite intruder. We are going to intercept the traffic with Burp Suite while using our browser to log in. After finding the login request in Burp Suite, we send it to intruder and input the user names and passwords for the payloads. After the attack finishes, we can check the responses to see which of them redirects us to the `/tracker` URL. We can now log in on the web page to get access to the Santa tracker. The flag is shown on the page.

## Day 4 - Santa's Watching
### Task 9
* Given the URL "http://shibes.xyz/api.php", what would the entire wfuzz command look like to query the "breed" parameter using the wordlist "big.txt" (assume that "big.txt" is in your current directory)

* Use GoBuster to find the API directory. What file is there?

* Fuzz the date parameter on the file you found in the API directory. What is the flag displayed in the correct post?
