+++
title = "Securely Booting Ubuntu Desktop"
date = "2021-04-10"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
cover = ""
tags = ["ubuntu", "linux", "crypto", "fde"]
keywords = ["ubuntu", "linux", "crypto", "fde"]
description = "[UEFI secure boot](https://wiki.ubuntu.com/UEFI/SecureBoot) has been supported out of the box by almost all major Linux distributions for many years. I always disabled it out of habit because I saw it as a hassle in the early days. Recently I've been learning more about the advantages of keeping secure boot enabled. The advantages really depend on how many steps you take to modify your installation."
showFullContent = false
+++

[UEFI secure boot](https://wiki.ubuntu.com/UEFI/SecureBoot) has been supported out of the box by almost all major Linux distributions for many years. I always disabled it out of habit because I saw it as a hassle in the early days. Recently I've been learning more about the advantages of keeping secure boot enabled. The advantages really depend on how many steps you take to modify your installation.

Ubuntu includes the `shim` bootloader which is signed by Microsoft and verified by your computer's firmware when you boot. Canonical signs all of their kernel and GRUB (bootloader) releases with their own key which is verified by the `shim` bootloader. As of kernel 5.4, which is included by default in Ubuntu 20.04, enabling secure boot also enables kernel lockdown. This [disables access to several kernel features](http://manpages.ubuntu.com/manpages/hirsute/man7/kernel_lockdown.7.html) which could be exploited by an attacker. Secure boot does not verify your initrd images which are generated on your PC and can change depending on your settings and installed software.

The boot process simplified:

`UEFI firmware -> shim -> GRUB -> kernel -> initramfs -> systemd`

Enabling full disk encryption in the Ubuntu installer only encrypts your system partition which contains all of the OS files and home directories. It does not encrypt your boot partition which contains all of your kernel and initrd files.

This guide will cover making sure secure boot and kernel lockdown are enabled, enabling the full disk encryption options in the Ubuntu installer, moving your boot files to an encrypted partition and adding key files to your boot and system partitions so that you only need to enter one password at boot.

## Contents
* [Installation](#installation)
* [Crypto setup](#crypto-setup)
* [Future improvements](#future-improvements)
* [Conclusion](#conclusion)

## Installation
This guide assumes that you're installing Ubuntu 20.04. The steps can be adapted for other Debian-based distributions, but kernel lockdown requires at least kernel 5.4.

Install Ubuntu and at the disk partitioning step check "Use LVM with the new Ubuntu installation" and "Encrypt the new Ubuntu installation for security."

![Enable FDE in the Ubuntu installer](/images/securely-booting-ubuntu-desktop/enable-fde.png)

Enter a password that you will need to enter at each boot.

![Enter FDE password](/images/securely-booting-ubuntu-desktop/enter-password.png)

Continue the installation and reboot your machine when it's completed.

Run `sudo apt update && sudo apt full-upgrade -y` and reboot when the upgrade is finished.

After installation is finished you can run `mokutil --sb-state` to verify that secure boot is enabled and `cat /sys/kernel/security/lockdown` to check the kernel lockdown status.

![Check secure boot and lockdown status](/images/securely-booting-ubuntu-desktop/verify-secure-boot.png)

## Crypto setup
The default Ubuntu install creates a FAT EFI partition, an unencrypted boot partition, and an encrypted system partition using LUKS2.

We need to encrypt the boot partition using LUKS1 since the version of GRUB included in Ubuntu 20.04 doesn't support LUKS2 encryption. LUKS1 uses PBKDF2 and LUKS2 uses argon2 which is memory-hard and makes it more expensive to crack your password. LUKS1 is still secure as long as your password is random and long.

The default Ubuntu FDE boot process simplified:

`UEFI firmware -> shim -> GRUB -> kernel -> initramfs -> *enter password* -> systemd`

After our modifications it changes to:

`UEFI firmware -> shim -> GRUB -> *enter password* -> kernel -> initramfs -> systemd`

Start by finding out where your boot drive is located

`sudo fdisk -l`

For example, I'm running Ubuntu in a VM for testing and mine is `/dev/vda`. It should have three partitions. Replace `vda` in the following steps with your actual drive.

Umount the EFI partition

`sudo umount /boot/efi`

Make a temporary directory to store the boot files while we format and encrypt the new partition.

`mkdir ~/boottemp`

Copy the boot files

`sudo cp -r /boot/* ~/boottemp`

Umount the boot partition

`sudo umount /boot`

Enable encryption on the boot partition.

`sudo cryptsetup luksFormat --type luks1 /dev/vda2`

Copy the UUID to your clipboard for later to make it easier.

`sudo blkid /dev/vda2`

![Copy the boot partition UUID](/images/securely-booting-ubuntu-desktop/boot-partition-uuid.png)

Open the encrypted partition (replace my UUID with yours)

`sudo cryptsetup luksOpen /dev/vda2 luks-808a8028-5ad2-494b-aadd-6bcf9a67f43d`

Format the boot partition

`sudo mkfs.ext4 /dev/mapper/luks-808a8028-5ad2-494b-aadd-6bcf9a67f43d`

Mount the boot partition

`sudo mount /dev/mapper/luks-808a8028-5ad2-494b-aadd-6bcf9a67f43d /boot`

Copy the boot files back to the boot partition

`sudo cp -r ~/boottemp/* /boot/`

Mount the EFI partition

`sudo mount /dev/vda1 /boot/efi`

Remove the temporary files directory

`sudo rm -r ~/boottemp`

Make a new directory for the key files

`sudo mkdir /etc/keys`

Set the permissions on the directory so that it's only readable by root

`sudo chmod 700 /etc/keys`

Create 64-byte random key files

`sudo dd if=/dev/urandom of=/etc/keys/boot.key bs=64 count=1`

`sudo dd if=/dev/urandom of=/etc/keys/system.key bs=64 count=1`

Set the permissions on the key files so they're only readable by root

`sudo chmod 400 /etc/keys/boot.key`

`sudo chmod 400 /etc/keys/system.key`

Add the boot.key file to the boot partition

`sudo cryptsetup luksAddKey /dev/vda2 /etc/keys/boot.key`

Enter your current password for the boot partition.

Add the system.key file to the system partition

`sudo cryptsetup luksAddKey /dev/vda3 /etc/keys/system.key`

Now enter your current password for the system partition to add the key file.

Modify the system encryption setup in /etc/crypttab to include the key file so that it will automatically be decrypted

`sudo sed -i 's+none+/etc/keys/system.key+' /etc/crypttab`

Add the line for the encrypted boot partition (change the UUID to the one previously used)

`echo 'boot_crypt UUID=808a8028-5ad2-494b-aadd-6bcf9a67f43d /etc/keys/boot.key luks,discard' | sudo tee -a /etc/crypttab`

Edit your /etc/fstab to change the previous /boot partition to the new encrypted one

`sudo sed -i 's+^UUID=.*/boot .*+/dev/mapper/boot_crypt /boot ext4 defaults 0 2+' /etc/fstab`

Add LUKS1 support to GRUB

`echo 'GRUB_ENABLE_CRYPTODISK=y' | sudo tee -a /etc/default/grub`

Add the key files to the initramfs image

`echo 'KEYFILE_PATTERN="/etc/keys/*.key"' | sudo tee -a /etc/cryptsetup-initramfs/conf-hook`

Make sure the new initramfs image file is only readable by root since it contains the LUKS key files

`echo 'UMASK=0077' | sudo tee -a /etc/initramfs-tools/initramfs.conf`

Reinstall GRUB with the added LUKS1 support

`sudo grub-install /dev/vda`

Update the initramfs images to contain the LUKS key files

`sudo update-initramfs -uk all`

Update the grub config

`sudo update-grub`

Reboot your machine. You will be prompted by GRUB to enter your LUKS password and then all of the other partitions will be unlocked automatically.

## Future improvements
I'd like to enable use of the TPM chip in the boot process on Linux. I haven't done enough research on how to enable this yet, but I will update the guide later.

## Conclusion
Enabling secure boot and encrypting your boot partition limits the attack surface of your machine at system boot. The default Ubuntu setup supports secure boot, but there is no way to encrypt your boot partition in the installer. Entering a few commands after the installation allows your boot files to be protected using full disk encryption.
