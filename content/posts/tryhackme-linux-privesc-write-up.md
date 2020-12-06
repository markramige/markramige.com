+++
title = "TryHackMe Linux PrivEsc Write-Up"
date = "2020-05-30"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["linux", "privesc", "suid", "authentication", "permissions", "cron", "nfs"]
keywords = ["linux", "privesc", "suid", "authentication", "permissions", "cron", "nfs"]
description = "This [TryHackMe room](https://tryhackme.com/room/linuxprivesc) gives us a vulnerable Debian virtual machine and demonstrates many different types of Linux privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can root Linux boxes is valuable. In addition, several tools are provided that show us how easy it is to automate checking for privesc vulnerabilities on Linux."
showFullContent = false
+++

This [TryHackMe room](https://tryhackme.com/room/linuxprivesc) gives us a vulnerable Debian virtual machine and demonstrates many different types of Linux privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can root Linux boxes is valuable. In addition, several tools are provided that show us how easy it is to automate checking for privesc vulnerabilities on Linux.

## Contents
* [Service Exploits](#service-exploits)
* [Weak File Permissions](#weak-file-permissions)
* [Sudo](#sudo)
* [Cron Jobs](#cron-jobs)
* [SUID/SGID Executables](#suidsgid-executables)
* [Passwords & Keys](#passwords--keys)
* [NFS](#nfs)
* [Kernel Exploits](#kernel-exploits)
* [Privilege Escalation Scripts](#privilege-escalation-scripts)

## Service Exploits
### Task 2
Follow the instructions given and you will end up with a setuid copy of bash to use to escalate your privileges.

## Weak File Permissions
### Task 3
* What is the root user's password hash?

It's the text you copied from the `/etc/shadow` file between the first and second colons.

* What hashing algorithm was used to produce the root user's password hash?

Run the john the ripper command given and the output will tell you what password hash was used. I had to `gunzip` the rockyou.txt.gz file on Kali before I could use it with `john`.

* What is the root user's password?

The password is given in the output of `john`.

### Task 4
Follow the instructions and you should end up being able to log in as `root` with your own password.

### Task 5
* Run the "id" command as the newroot user. What is the result?

Follow the instructions and we can now log in as root using the password generated with `openssl` and inserted into `/etc/passwd`.

## Sudo
### Task 6
* How many programs is "user" allowed to run via sudo?

Count up the number of programs listed at the bottom of the output from `sudo -l`

* One program on the list doesn't have a shell escape sequence on [GTFOBins](https://gtfobins.github.io). Which is it?

Search all the programs on [GTFOBins](https://gtfobins.github.io) and find the one which doesn't provide any ways to escalate.

### Task 7
Follow the instructions and you will end up with a root shell by abusing LD_PRELOAD to escalate privileges.

## Cron Jobs
* What is the value of the PATH variable in /etc/crontab?

### Task 8
Follow the instructions given and you will end up with a root shell using netcat and cron.

### Task 9
* What is the value of the PATH variable in /etc/crontab?

Look at the top of /etc/crontab to answer this. This is a problem because cron executes with root permissions, so if a normal user is able to drop a script or program in cron's path then it will be executed as root.

### Task 10
Follow the instruction in the task text. You will use the fact that the `tar` command is run by cron with a wildcard `*` character. This allows you to generate a reverse shell binary that will connect to your netcat listener and give you root access.

## SUID/SGID Executables
### Task 11
For this task we'll find that `exim` is SUID and the version included on this box is [vulnerable](https://www.exploit-db.com/exploits/39535). The script to exploit this vulnerability is already included on the box so we just have to run it to get root access.

### Task 12
In this task we execute a SUID executable which is trying to open a shared object file in our home directory. We create an object file which spawns a root shell.

### Task 13
This task takes advantage of a SUID executable that attempts to execute a program without specifying an absolute path. We can then create an executable in our path that the SUID program will call and give us a root shell.

### Task 14
Similar to task 13, we have a SUID executable. This time it does specify an absolute path, but our version of Bash is not updated so it allows us to define a shell function that resembles that path and executable which will then run instead of the actual executable and give us a root shell.

### Task 15
This task takes advantage of another flaw in older versions of Bash. We're going to run a SUID executable with bash debugging enabled which executes a copy of the bash executable with root privileges.

## Passwords & Keys
### Task 16
* What is the full mysql command the user executed?
The user should have executed the `mysql` command without the password included and it would've prompted for a password that would not be included in the `.history` file.

### Task 17
It's important to set the correct permissions on config files, especially those that contain credentials. This file is world readable which makes it easy to view the root password as a normal user.

### Task 18
Usually `ssh` will give you an error if your keys do not have the correct permissions. In this case, a user backed up the root SSH keys without the correct permissions and the file ended up being world readable.

## NFS
### Task 19
We have root on our Kali box, so we're going to create a SUID executable that will also have root permissions on our Debian VM since the /tmp directory doesn't have root squashing enabled.

## Kernel Exploits
### Task 20
Our Debian VM is vulnerable to the Dirty COW vulnerability since it is running a kernel <4.8.3 and hasn't been patched. We can exploit this vulnerability to gain a root shell.

## Privilege Escalation Scripts
### Task 21
All three of these tools check for most of the vulnerabilities that we've experimented with. Automated scanning tools make it extremely easy for an attacker or penetration tester to find vulnerabilities on a Linux box.
