+++
title = "TryHackMe OWASP Top 10 Write-up"
date = "2020-07-25"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["appsec", "owasp", "xss", "idor", "injection"]
keywords = ["appsec", "owasp", "xss", "idor", "injection"]
description = "[This challenge](https://tryhackme.com/room/owasptop10) on TryHackMe was initially released over a period of ten days covering one of the OWASP Top Ten vulnerabilities per day. In includes an introduction and explanation of each vulnerability type and one or multiple exercises for each. It is beginner friendly."
showFullContent = false
+++

[This challenge](https://tryhackme.com/room/owasptop10) on TryHackMe was initially released over a period of ten days covering one of the OWASP Top Ten vulnerabilities per day. In includes an introduction and explanation of each vulnerability type and one or multiple exercises for each. It is beginner friendly.

## Contents
* [Day 1 - Injection](#day-1---injection)
* [Day 2 - Broken Authentication](#day-2---broken-authentication)
* [Day 3 - Sensitive Data Exposure](#day-3---sensitive-data-exposure)
* [Day 4 - XML External Entities](#day-4---xml-external-entities)
* [Day 5 - Broken Access Control](#day-5---broken-access-control)
* [Day 6 - Security Misconfiguration](#day-6---security-misconfiguration)
* [Day 7 - Cross-site Scripting (XSS)](#day-7---cross-site-scripting-(xss))
* [Day 8 - Insecure Deserialization](#day-8---insecure-deserialization)
* [Day 9 - Using Components with Known Vulnerabilities](#day-9---using-components-with-known-vulnerabilities)
* [Day 10 - Insufficient Logging and Monitoring](#day-10---insufficient-logging-and-monitoring)

## Day 1 - Injection
### Task 5
*Visit `http://MACHINE_IP/evilshell.php` to complete the following:*
* What strange text file is in the website root directory?

`ls /var/www/html`

* How many non-root/non-service/non-daemon users are there?

`cat /etc/passwd` and look for regular user accounts

* What user is this app running as?

`whoami`

* What is the user's shell set as?

`grep USERNAME /etc/passwd` (replace USERNAME with the previous answer's user name)

* What version of Ubuntu is running?

`lsb_release -a`

* Print out the MOTD. What favorite beverage is shown?

`cat /etc/update-motd.d/00-header`

## Day 2 - Broken Authentication
### Task 7
*Visit `http://MACHINE_IP:8888` to complete the following:*
* What is the flag that you found in darren's account?

Follow the instructions to create an account " darren" and set the password. Log in as that account with your password and the flag will be shown.

* Now try to do the same trick and see if you can login as arthur.


* What is the flag that you found in arthur's account?

Follow the instructions to create an account " arthur" and set the password. Log in as that account with your password and the flag will be shown.

## Day 3 - Sensitive Data Exposure
### Task 11
*Visit `http://MACHINE_IP` to complete the following:*
* What is the name of the mentioned directory?

Look at the source of the /login page and the answer is commented out.

* Navigate to the directory you found in question one. What file stands out as being likely to contain sensitive data?

There is a database file listed in the directory.

* Use the supporting material to access the sensitive data. What is the password hash of the admin user?

Running the `file` command on the database file reveals that it is a sqlite3 file. We can now use the `sqlite3` command to access the sensitive data. We use the instructions in Task 9 to get the password hash.

* Crack the hash. What is the admin's plaintext password?

Use [Crackstation](https://crackstation.net/) to crack the password hash.

* Login as the admin. What is the flag?

Use the credentials from the cracked hash to get the flag.


## Day 4 - XML External Entities
### Task 16
*Visit `http://MACHINE_IP` to complete the following:*
* Try to display your own name using any payload.

I used a modified payload from Task 15 to display my name:
```
<!DOCTYPE replace [<!ENTITY name "Mark"> ]>
 <userInfo>
  <firstName>&name;</firstName>
 </userInfo>
 ```

* See if you can read the /etc/passwd

We can use the payload from Task 15 as is:
```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

* What is the name of the user in /etc/passwd

The user with id 1000 is the one they're looking for.

* Where is falcon's SSH key located?

We can modify the previous payload slightly to read a different file where we think the SSH key is located:
```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///home/falcon/.ssh/id_rsa'>]>
<root>&read;</root>
```

* What are the first 18 characters for falcon's private key

We already successfully read the private key in the previous answer so we just copy and paste the first 18 characters.

## Day 5 - Broken Access Control
### Task 18
*Visit `http://MACHINE_IP` to answer the following:*
* Look at other users notes. What is the flag?

After logging in, the URL changes to `http://MACHINE_IP/note.php?note=1`. We can change the note parameter to different numbers to access other users' notes.

## Day 6 - Security Misconfiguration
### Task 19
*Visit `http://MACHINE_IP` to answer the following:*
* Hack into the webapp, and find the flag!

This challenge focuses on default credentials. We can search the web for the source code for this web app and see if there is a default user account that hasn't been deleted or whose password hasn't been changed.

We find the source on [Github](https://github.com/NinjaJc01/PensiveNotes). The documentation gives us the default user name and password.

Logging in with those credentials gives us the flag.

## Day 7 - Cross-site Scripting (XSS)
### Task 20
* Navigate to `http://MACHINE_IP/` in your browser and click on the "Reflected XSS" tab on the navbar; craft a reflected XSS payload that will cause a popup saying "Hello".

We can submit `alert('Hello')` in the search box to demonstrate reflected XSS because there is no filtering on the search input.

* On the same reflective page, craft a reflected XSS payload that will cause a popup with your machines IP address.

First we need to find out how to get the hostname with javascript. [w3schools](https://www.w3schools.com/js/js_window_location.asp) comes through with the first result.

We try `alert(window.location.hostname)` in the search box and the flag is returned.

* Now navigate to `http://MACHINE_IP/stored` in your browser and make an account.
  Then add a comment and see if you can insert some of your own HTML.

We can use `<h1>test</h1>` in the comment box to insert HTML on the page and the flag is returned.

* On the same page, create an alert popup box appear on the page with your document cookies.

We check [w3schools](https://www.w3schools.com/js/js_cookies.asp) again for how to get cookies in Javascript. The payload `<script>alert(document.cookie);</script>` returns our cookies with an alert box and gives us the flag.

* Change "XSS Playground" to "I am a hacker" by adding a comment and using Javascript.

We use [w3schools](https://www.w3schools.com/js/js_htmldom_html.asp) again to find out how to change text on the page using javascript.

Inspecting the page reveals that we need to change `<span id="thm-title">XSS Playground</span>` to `<span id="thm-title">I am a hacker</span>`.

We can use `<script>document.getElementById("thm-title").innerHTML = "I am a hacker";</script>` to change the text and get the flag.

## Day 8 - Insecure Deserialization
### Task 21
* Who developed the Tomcat application?

Google should give us the answer here.

* What type of attack that crashes services can be performed with insecure deserialization?

The answer is given in the Task text.

### Task 22
* Select the correct term of the following statement:
if a dog was sleeping, would this be:
A) A State
B) A Behaviour

Read the task text and understand the difference between state and behavior.

### Task 23
* What is the name of the base-2 formatting that data is sent across a network as?

Read the task text to find the answer.

### Task 24
* If a cookie had the path of webapp.com/login , what would the URL that the user has to visit be?

Read the task text to find the answer.

* What is the acronym for the web technology that Secure cookies work over?

Again the answer is in the text.

### Task 25
*Visit `http://MACHINE_IP` to answer the following:*
* 1st flag (cookie value)

We can use the instructions in the task text to view the cookies after we log in. The `sessionId` cookie is base64 encoded. We can visit [CyberChef](https://gchq.github.io/CyberChef/) and use the `From Base64` recipe on the `sessionId` value to get the flag.

* 2nd flag (admin dashboard)

We change the `userType` cookie value to `admin` and navigate to `http://MACHINE_IP/admin` to get the admin flag.

### Task 26
* flag.txt

Follow the instructions in the task text to get a remote shell. Next run `ls $HOME` to find the flag file and `cat $HOME/flag.txt` to get the flag text.

## Day 9 - Using Components with Known Vulnerabilities
### Task 29
* How many characters are in /etc/passwd (use wc -c /etc/passwd to get the answer)

Googling for the name of the web application, `online bookstore` leads us to an [exploit](https://www.exploit-db.com/exploits/47887). We copy and paste the code into a file.

Run the file `python3 test.py http://MACHINE_IP`

Run the command given at the prompt `wc -c /etc/passwd`

## Day 10 - Insufficient Logging and Monitoring
### Task 30
* What IP address is the attacker using?
Look for unauthorized login attempts.

* What kind of attack is being carried out?
The attacker is repeatedly trying different credentials to log in.
