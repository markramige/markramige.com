+++
title = "TryHackMe WebAppSec 101 Write-Up"
date = "2020-03-22"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["weak credentials", "idor", "sql injection", "logic flaws", "directory traversal"]
keywords = ["weak credentials", "idor", "sql injection", "logic flaws", "directory traversal"]
description = "[This TryHackMe room](https://tryhackme.com/room/webappsec101) contains a web application with multiple vulnerabilities. The tasks step us through different types of vulnerabilities with links to authoritative resources to learn more about each type. There is a good combination of hand-holding and do-it-yourself. The vulnerabilities include weak credentials, cross-site scripting (xss), command and sql injection, idor, directory traversal, and logic flaws."
showFullContent = false
+++

[This TryHackMe room](https://tryhackme.com/room/webappsec101) contains a web application with multiple vulnerabilities. The tasks step us through different types of vulnerabilities with links to authoritative resources to learn more about each type. There is a good combination of hand-holding and do-it-yourself. The vulnerabilities include weak credentials, cross-site scripting (xss), command and sql injection, idor, directory traversal, and logic flaws.

## Contents
* [Task 2 - Walking through the application](#task-2---walking-through-the-application)
* [Task 4 - Authentication](#task-4---authentication)
* [Task 5 - Cross Site Scripting (XSS)](#task-5---cross-site-scripting-xss)
* [Task 6 - Injection](#task-6---injection)
* [Task 7 - Miscellaneous & Logic Flaws](#task-7---miscellaneous--logic-flaws)

## Task 2 - Walking through the application
We can proxy the traffic through Burp Suite to view all of the response headers. For this web application, it sends a `Server` and `X-Powered-By` header with every response.

For the server specifically, we can also force it to produce a 404 error by visiting `http://MACHINE_IP/asdf` (or any other nonexisting page) and the server name and version is given on the 404 error page. We could also run `nmap -A -T4 MACHINE_IP` to get the server name and version.

* What version of Apache is being used?

Check the `Server` response header in Burp Suite.

* What language was used to create the website?

Check the `X-Powered-By` reponse header.

* What version of the language is used?

Check the `X-Powered-By` reponse header.

## Task 4 - Authentication
* What is the admin username?

There's a link to the admin area at the bottom of each page. Click on the link and try to guess the user name.

* What is the admin password?

Try to guess the password using common passwords.

An alternative to guessing (if this challenge was more difficult) would be to use Burp Suite Intruder to brute force the password using a word list.

* What is the name of the cookie that can be manipulated?

A cookie is set when logging into the admin page. The value is a single digit that can be easily manipulated to steal a session.

* What is the username of a logged on user?

For this we want to brute force the user creation page (`http://MACHINE_IP/users/register.php`) since it will tell us when an account already exists.

We can use [OWASP ZAP](https://owasp.org/www-project-zap/) to fuzz the user names since the Burp Suite Community Edition is slow at brute forcing.

We're going to use the [SecLists](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/Names/names.txt) user name list.

We're registering an account with each user name that gets submitted, so what we're looking for in the output is the name that can't be registered because it already exists.

* What is the corresponding password to the username?

Try guessing first based on what you know about the user name and user. If you can't guess the password, try brute forcing it with a [word list](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

## Task 5 - Cross Site Scripting (XSS)

Use the first payload on the linked page to test for XSS.

`<SCRIPT SRC=http://xss.rocks/xss.js></SCRIPT>`

* Test for XSS on the search bar

This input is vulnerable to reflected XSS.

* Test for XSS on the guestbook page

This input is vulnerable to stored XSS.

## Task 6 - Injection

* Perform command injection on the check password field

If we test the field, we get back `The command "grep ^test$ /etc/dictionaries-common/words" was used to check if the password was in the dictionary.` which is very handy!

We just need to end the grep command and insert our own command after it.

`'';ls;` should work for any command that we have permissions to run. This will hang the application.

* Check for SQLi on the application

The login form (http://MACHINE_IP/users/login.php) is vulnerable to SQLi. Try testing [payloads](https://medium.com/@ismailtasdelen/sql-injection-payload-list-b97656cfd66b) to see what is possible.

## Task 7 - Miscellaneous & Logic Flaws

* Find a parameter manipulation vulnerability

The sample page linked from the home page at `http://MACHINE_IP/users/sample.php?userid=1` contains the `userid` parameter that we can easily change to view additional photos.

* Find a directory traversal vulnerability

The file name field on the picture upload page (`http://MACHINE_IP/pictures/upload.php`) allows us to exploit directory traversal.

* Find a forceful browsing vulnerability

We can visit `http://MACHINE_IP/upload/` and view all images that have been uploaded by all users.

* Logic flaw: try to get an item for free

We can fuzz the coupon code field on the cart review page (`http://MACHINE_IP/cart/review.php`). Unsuccessful attempts give us a 303 status code.
