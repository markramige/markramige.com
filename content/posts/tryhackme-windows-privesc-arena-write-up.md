+++
title = "TryHackMe Windows PrivEsc Arena Write-Up"
date = "2020-05-10"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["windows", "privesc", "registry", "authentication", "permissions", "services", "rdp"]
keywords = ["windows", "privesc", "registry", "authentication", "permissions", "services", "rdp"]
description = "The Windows PrivEsc Arena [TryHackMe room](https://tryhackme.com/room/windowsprivescarena) includes a vulnerable Windows 7 virtual machine and demonstrates several types of privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can gain elevated privileges on a Windows machine is valuable. Also, many of the tasks show us how to use features of Metasploit to exploit Windows machines."
showFullContent = false
+++

The Windows PrivEsc Arena [TryHackMe room](https://tryhackme.com/room/windowsprivescarena) includes a vulnerable Windows 7 virtual machine and demonstrates several types of privilege escalation techniques. There aren't many challenges included in the room, but just knowing how many different ways attackers can gain elevated privileges on a Windows machine is valuable. Also, many of the tasks show us how to use features of Metasploit to exploit Windows machines.

## Contents
* [Registry Escalation](#registry-escalation)
* [Service Escalation](#service-escalation)
* [Privilege Escalation](#privilege-escalation)
* [Potato Escalation](#potato-escalation)
* [Password Mining Escalation](#password-mining-escalation)

## Registry Escalation
### Task 3
In this task we notice that `C:\Program Files\Autorun Program\program.exe` is set to run at startup with administrator privileges. We also notice that the `Everyone` user group has `FILE_ALL_ACCESS` permission on `program.exe`. This means we can replace the file with our own version and have that run at startup instead. Next we use `Metasploit` to open a listener on our Kali box and generate a reverse shell to place on the Windows system as `program.exe`. The next time we log on the reverse shell will run and we will be connected with administrator privileges.

### Task 4
We check the registry for the `AlwaysInstallElevated` key and notice that it is set to `1`. This means anything we install will have elevated privileges even if we install it as a normal user. We can generate a reverse shell installer with Metasploit and install it on our Windows machine. This will automatically connect to our listener and give us elevated permissions after installation.

## Service Escalation
### Task 5
The ability to control the registry entry for a Windows service means we can replace that executable file that is run when the service starts. In this task, we create a custom service executable that adds our normal user to the `administrators` group. After we start the service, our user will have elevated privileges.

### Task 6
This task is similar to Task 5, but we don't need registry permissions. We check the `filepermsvc` executable and notice that the `Everyone` group has `FILE_ALL_ACCESS` permissions. This means we can just replace the executable with the same one we used in Task 5 and then start the service.

### Task 8
Using `procmon`, we notice that `dllsvc` is calling `C:\Temp\hijackme.dll`, but it is not found. We know that `C:\Temp` is a writeable location, so we generate a reverse shell using Metasploit and copy it to that location. We can then start `dllsvc` and our reverse shell will connect.

### Task 9
Having permissions to configure a Windows service means we can change the command that runs when the service starts. We're going to change the `daclsvc` to run `net localgroup administrators user /add`. This will add our user to the administrators group so that we have elevated privileges.

### Task 10
When looking at the details of the `unquotedsvc`, we notice that the `BINARY_PATH_NAME` is unquoted. Because there's a space in the path and we can write to that directory, we can make Windows execute `C:\Program Files\Unquoted Path Service\common.exe` instead of `C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe`

## Privilege Escalation
### Task 7
We notice that as a normal user we can write to the `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup` directory. Any programs that we add here will be run with administrator privileges. We create a reverse shell and place it in that directory and the log back in to connect to our listener with elevated privileges.

### Task 14
In this task we're using Metasploit to suggest exploits to use based on our Windows machine version and update status. It suggests using `exploit/windows/local/ms16_014_wmi_recv_notif`. We then run the exploit and gain elevated privileges in our reverse shell.

## Potato Escalation
### Task 11
The [Potato exploit](https://github.com/foxglovesec/Potato) allows us to run an abritrary command with elevated privileges. For this task we're going to use Potato to run `net localgroup administrators user /add` which gives our normal user administrator privileges.

## Password Mining Escalation
### Task 12
This task has us look at a config file for possible secrets. We notice that the `<Password>` property value is base64 encoded. We can decode it to find the account password in clear text.

### Task 13
We're going to capture a memory dump of the running `iexplore.exe` process. Running `strings` on the dump file allows us to extract base64 encoded passwords that have been sent to a website.
