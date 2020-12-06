+++
title = "Using SSH on Windows 10 and WSL 2"
date = "2020-09-20"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
#cover = ""
tags = ["wsl2", "ssh", "git", "linux", "windows"]
keywords = ["wsl2", "ssh", "git", "linux", "windows"]
description = "Since WSL was introduced several years ago, I've been using the same process for access my machines remotely. I'd run `apt-get install openssh-server` in WSL and open TCP port 22 in Windows Firewall. WSL 2 adds the extra step of needing to [forward a port to the WSL VM](https://www.hanselman.com/blog/HowToSSHIntoWSL2OnWindows10FromAnExternalMachine.aspx). I decided to try using OpenSSH directly since it is now available [natively on Windows](https://github.com/PowerShell/Win32-OpenSSH)."
showFullContent = false
+++

Since WSL was introduced several years ago, I've been using the same process for access my machines remotely. I'd run `apt-get install openssh-server` in WSL and open TCP port 22 in Windows Firewall. WSL 2 adds the extra step of needing to [forward a port to the WSL VM](https://www.hanselman.com/blog/HowToSSHIntoWSL2OnWindows10FromAnExternalMachine.aspx). I decided to try using OpenSSH directly since it is now available [natively on Windows](https://github.com/PowerShell/Win32-OpenSSH).

## Contents
* [Install sshd, ssh, and ssh-agent](#install-sshd-ssh-and-ssh-agent)
* [Run sshd and ssh-agent at boot](#run-sshd-and-ssh-agent-at-boot)
* [Start sshd and ssh-agent now](#start-sshd-and-ssh-agent-now)
* [Create ssh key and add to ssh-agent](#create-ssh-key-and-add-to-ssh-agent)
* [Add public keys to authorized_keys and set correct permissions](#add-public-keys-to-authorized_keys-and-set-correct-permissions)
* [Disable password login and restart sshd](#disable-password-login-and-restart-sshd)
* [Make Git for Windows use ssh-agent](#make-git-for-windows-use-ssh-agent)
* [Set up WSL distributions to use ssh-agent](#set-up-wsl-2-distributions-to-use-ssh-agent)
* [Future improvements](#future-improvements)
* [Conclusion](#conclusion)

`*Open a PowerShell prompt with administrator privileges*`

## Install sshd, ssh, and ssh-agent

```
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

## Run sshd and ssh-agent at boot
```
Set-Service -Name sshd -StartupType 'Automatic'
Set-Service -Name ssh-agent -StartupType 'Automatic'
```

## Start sshd and ssh-agent now
```
Start-Service sshd
Start-Service ssh-agent
```

## Create SSH key and add to ssh-agent
Ignore the `-t ed25519` if you need to connect to servers running older OSes

```
ssh-keygen -t ed25519
ssh-add
ssh-add -L
```

## Add public keys to authorized_keys and set correct permissions
Windows sshd will read two different authorized_keys files depending on if your user account is an administrator.

The location for normal users is `%userprofile%\.ssh\authorized_keys`

The location for administrators is `%programdata%\ssh\administrators_authorized_keys`

The `administrators_authorized_keys` file will need special permissions set before sshd will use it

Copy any public keys you want to grant access to your computer to the correct file and then set the administrators_authorized_keys file permissions if needed.

```
icacls administrators_authorized_keys /inheritance:r
icacls administrators_authorized_keys /grant SYSTEM:`(F`)
icacls administrators_authorized_keys /grant BUILTIN\Administrators:`(F`)
```

## Disable password login and restart sshd

```
Stop-Service sshd
```

Copy C:\ProgramData\ssh\sshd_config to your desktop and edit

```
#PasswordAuthentication yes
PasswordAuthentication no
```
Move the file back to %programdata%\ssh and then set the correct permissions

```
Get-Acl sshd.pid | Set-Acl sshd_config
Start-Service sshd
```

## Make Git for Windows use ssh-agent
Search for `env` in start and choose the `Edit environment variables for your account` option.
![Search for env on the start menu and choose Edit the system environment variables](/images/using-ssh-on-windows-10-and-wsl-2/start-menu-edit-environment-variables.png)

Create a new variable named `GIT_SSH` with a value of `C:\Windows\System32\OpenSSH\ssh.exe`
![Add the GIT_SSH variable](/images/using-ssh-on-windows-10-and-wsl-2/windows-add-git-ssh-environment-variable.png)

## Set up WSL 2 distributions to use ssh-agent
Download [wsl-ssh-agent.7z](https://github.com/rupor-github/wsl-ssh-agent/releases/download/v1.4.2/wsl-ssh-agent.7z) and extract `npiperelay.exe` to a new directory called `.wsl` in your Windows user's home folder.

In your WSL distribution, run `ln -s /mnt/c/Users/YOURWINDOWSUSERNAME ~/winhome`

Create a `.ssh` folder and `chmod 700 ~/.ssh` if it doesn't already exist.

Run `sudo apt install socat`

Add this snippet to the `.bashrc` file in your WSL distribution.

```bash
export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock
ss -a | grep -q $SSH_AUTH_SOCK
if [ $? -ne 0   ]; then
    rm -f $SSH_AUTH_SOCK
    ( setsid socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:"$HOME/winhome/.wsl/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent",nofork & ) >/dev/null 2>&1
fi
```

## Future improvements
I would like to sync up my ssh config file between Windows and WSL. Symlinking the file inside WSL doesn't work because it needs 600 permissions or ssh will give you an error.

## Conclusion
I like the added benefit of being able to run native Windows commands via SSH and being able to access multiple WSL distributions with the same SSH server. I think if you are already using sshd inside a WSL distribution and don't need these benefits, then it's not worth switching to native Windows OpenSSH.
