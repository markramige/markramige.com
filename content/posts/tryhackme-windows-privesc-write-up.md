+++
title = "TryHackMe Windows PrivEsc Write-Up"
date = "2020-06-30"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["windows", "privesc", "registry", "authentication", "permissions", "services", "rdp"]
keywords = ["windows", "privesc", "registry", "authentication", "permissions", "services", "rdp"]
description = "This [TryHackMe room](https://tryhackme.com/room/windows10privesc) gives us a vulnerable Windows Server 2019 virtual machine and demonstrates many different types of Windows privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can gain elevated privileges on a Windows machine is valuable. In addition, several tools are provided that show us how easy it is to automate checking for privesc vulnerabilities on Windows."
showFullContent = false
+++

This [TryHackMe room](https://tryhackme.com/room/windows10privesc) gives us a vulnerable Windows Server 2019 virtual machine and demonstrates many different types of Windows privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can gain elevated privileges on a Windows machine is valuable. In addition, several tools are provided that show us how easy it is to automate checking for privesc vulnerabilities on Windows.

## Contents
* [Service Exploits](#service-exploits)
* [Registry](#registry)
* [Passwords](#passwords)
* [Scheduled Tasks](#scheduled-tasks)
* [Insecure GUI Apps](#insecure-gui-apps)
* [Startup Apps](#startup-apps)
* [Token Impersonation](#token-impersonation)
* [Privilege Escalation Scripts](#privilege-escalation-scripts)

## Service Exploits
### Task 3
First off, we're checking permission on the `daclsvc` service. We notice that we have `SERVICE_CHANGE_CONFIG` permissions and that the service runs with SYSTEM privileges. All we have to do is change the `BINARY_PATH_NAME` of the service to make it run our reverse shell when the service starts.

### Task 4
Similar to Task 3, we check the permissions of the directory containing the `unquotedsvc` service and notice that we can write to the directory. Since the `BINARY_PATH_NAME` is unquoted, we copy our reverse shell to the directory and rename it to `common.exe`. Since Windows will find `common.exe` before `Common Files` it executes it instead of the intended service.

### Task 5
We find that the registry entry for the `regsvc` service has weak permissions. We can change the location of the service executable in the registry so that it runs our reverse shell.

### Task 6
The `filepermsvc` executable located at `C:\Program Files\File Permissions Service\filepermservice.exe` is writeable by all users. All we have to do is replace it with our reverse shell executable and start the service.

## Registry
### Task 7
In this task we're checking the registry for programs that are set to run automatically at boot. These will be run with admin privileges. We notice that one program is writeable by all users. We can replace this program with our reverse shell and restart the machine to gain admin access.

### Task 8
We query the registry and find that the `AlwaysInstallElevated` key is enabled. We can generate an installer that will automatically run with elevated privileges when it is installed.

## Passwords
### Task 9
We can query the registry for the string `password` to find user credentials. We find admin AutoLogin credentials and use the `winexe` command in Kali to log on to the machine.

### Task 10
A local admin user has saved their login credentials on the machine. We use the `cmdkey /list` command to check for credentials and then the `runas /savecred` command to take advantage of the saved credentials to run our reverse shell with admin privileges.

### Task 11
We have access to backup copies of the `SAM` and `SYSTEM` files for our machine. We can use `creddump7` to extract the NTLM hashes of the users on the system. We then use `hashcat` to crack the admin password using a wordlist.

### Task 12
Since we have the NTLM hash, we can now use `pass the hash` to login to the system without even needing to crack the hash to obtain the password.

## Scheduled Tasks
### Task 13
The machine has a task set to run a PowerShell script every minute with SYSTEM privileges. In addition, we have permission to write to this file. All we have to do is add a line to the script to run our reverse shell executable.

## Insecure GUI Apps
### Task 14
Pasting in a file location in the navigation input of the file open dialog bypasses the file type check that the application usually performs when opening a file. Since the application is running with admin privileges, it will also run our `cmd.exe` with admin privileges.

## Startup Apps
### Task 15
We have permissions to write to the admin startup directory. We're going to create a new shortcut to our reverse shell executable in the startup directory so that it will run the next time the admin user logs on.

## Token Impersonation
### Task 16
For this task we're exploting a vulnerability in Windows to get local privilege execution. It turns out that if our user has `SeImpersonate` or `SeAssignPrimaryToken` privileges, we can use those to gain `SYSTEM` privileges.

### Task 17
We're again abusing `SeImpersonatePrivilege` with the PrintSpoofer vulnerability to escalate local access to `SYSTEM`.

## Privilege Escalation Scripts
### Task 18
All four of these tools check for most of the vulnerabilities that we've experimented with. Automated tools make it extremely easy for an attacker or penetration tester to find vulnerabilities on a Windows machine.
