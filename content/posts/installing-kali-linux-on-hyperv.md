+++
title = "Installing and Optimizing Kali Linux on Hyper-V"
date = "2020-10-04"
author = "Mark Ramige"
authorTwitter = "markramige" #do not include @
#cover = ""
tags = ["kali", "linux", "hyper-v", "windows"]
keywords = ["kali", "linux", "hyper-v", "windows"]
description = "Kali Linux runs great on Windows if you’re using VirtualBox or VMware Workstation. You might have tried installing it on Hyper-V so that you can run WSL 2 distributions at the same time and experienced how poorly Hyper-V handles desktop Linux. [Microsoft is working to fix this](https://github.com/microsoft/linux-vm-tools) by enabling enhanced session mode on desktop linux distributions. I’ll show you how to set up Kali Linux on Hyper-V so that it runs faster and also to enable copying and pasting between the VM and host (including files)."
showFullContent = false
+++

Kali Linux runs great on Windows if you’re using VirtualBox or VMware Workstation. You might have tried installing it on Hyper-V so that you can run WSL 2 distributions at the same time and experienced how poorly Hyper-V handles desktop Linux. [Microsoft is working to fix this](https://github.com/microsoft/linux-vm-tools) by enabling enhanced session mode on desktop linux distributions. I’ll show you how to set up Kali Linux on Hyper-V so that it runs faster and also to enable copying and pasting between the VM and host (including files).

## Contents
* [Hyper-V network setup](#hyper-v-network-setup)
* [Create a new virtual machine](#create-a-new-virtual-machine)
* [Create a virtual hard disk and enable enhanced session](#create-a-virtual-hard-disk-and-enable-enhanced-session)
* [Open and adjust VM settings](#open-and-adjust-vm-settings)
* [Create partition table and format partitions](#create-partition-table-and-format-partitions)
* [Install Kali Linux](#install-kali-linux)
* [Install linux-vm-tools](#install-linux-vm-tools)
* [Adjust XFCE settings](#adjust-xfce-settings)
* [Future improvements](#future-improvements)
* [Conclusion](#conclusion)

`*Open a PowerShell prompt with administrator privileges*`

## Hyper-V network setup
We need to create a new network adapter called “External” that is bridged to your wifi or ethernet network adapter.

Run `Get-NetAdapter` to get a list of your network adapters. Note the name of the one that connects to the Internet.

Run `New-VMSwitch -Name "External" -AllowManagementOS $True -NetAdapterName "YOURADAPTERNAME"` and edit the name of your network adapter.

You will be disconnected from the network when the adapter is added. If your connection isn’t working after a few seconds, then the settings from your real adapter didn’t copy over correctly to the new “External” adapter. Go and edit them manually to the correct values. My DNS server settings didn’t copy over and I had to add them manually.

## Create a new virtual machine
Open up the Hyper-V Manager by running `C:\Windows\System32\virtmgmt.msc`

![Hyper-V Manager](/images/installing-kali-linux-on-hyperv/hyper-v-manager.png)

Click on “New” and then “Virtual Machine”

Choose a name for the new VM.

![Create VM name](/images/installing-kali-linux-on-hyperv/new-vm-name.png)

Select “Generation 2”

![Create VM select generation 2](/images/installing-kali-linux-on-hyperv/new-vm-generation-2.png)

Choose how much memory you want to give the machine on startup. Since we’re also selecting “Dynamic Memory” it can go higher depending on what you set as the maximum later.

![Create VM assign memory](/images/installing-kali-linux-on-hyperv/new-vm-assign-memory.png)

Choose the External adapter that we created earlier.

![Create VM select external connection](/images/installing-kali-linux-on-hyperv/new-vm-network-connection.png)

We’re going to create a virtual hard disk manually and attach it later.

![Create VM attach VHD later](/images/installing-kali-linux-on-hyperv/new-vm-attach-vhd-later.png)

## Create a virtual hard disk and enable enhanced session
Since we’re creating a dynamic vhdx file, we want to lower the block size so that it [will use up less real space](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/best-practices-for-running-linux-on-hyper-v). The path and file name can be anything you want. You can set the SizeBytes (maximum size of the disk) value to anything over 32GB.

`New-VHD -Path C:\VHDs\Kali.vhdx -SizeBytes 128GB -Dynamic -BlockSizeBytes 1MB`

Run `Set-VM -VMName Kali -EnhancedSessionTransportType HvSocket` to enable enhanced session mode.

## Open and adjust VM settings
![VM open settings](/images/installing-kali-linux-on-hyperv/vm-open-settings.png)

Disable secure boot

![VM settings turn off secure boot](/images/installing-kali-linux-on-hyperv/vm-settings-turn-off-secure-boot.png)

Set the maximum RAM that you can assign to this VM.

![VM settings memory size](/images/installing-kali-linux-on-hyperv/vm-settings-memory-size.png)

Set the number of processors you want to give the VM.

![VM settings processors](/images/installing-kali-linux-on-hyperv/vm-settings-processors.png)

I always turn off automatic checkpoints because I like to do them manually.

![VM settings checkpoints](/images/installing-kali-linux-on-hyperv/vm-settings-checkpoints.png)

I don’t want this machine starting at boot.

![VM settings startup](/images/installing-kali-linux-on-hyperv/vm-settings-no-automatic-start.png)

Browse to the vhdx file you created earlier.

![VM settings select VHD](/images/installing-kali-linux-on-hyperv/vm-settings-select-vhd.png)

Add a DVD drive for installation.

![VM settings add DVD drive](/images/installing-kali-linux-on-hyperv/vm-settings-add-dvd-drive.png)

Select the Kali live iso you downloaded earlier.

![VM settings select Kali live iso](/images/installing-kali-linux-on-hyperv/vm-settings-select-kali-live-iso.png)

Set the boot order so that it will boot from the iso automatically.

![VM Settings set boot order](/images/installing-kali-linux-on-hyperv/vm-settings-set-boot-order.png)

## Create partition table and format partitions
Boot the default option in the menu

![Kali live boot menu](/images/installing-kali-linux-on-hyperv/kali-live-boot-menu.png)

Open a terminal and run `sudo gdisk /dev/sda`

Type `o` to create a new partition table

![Kali live gdisk create partition table](/images/installing-kali-linux-on-hyperv/kali-live-gdisk-new.png)

Type `n` and for the last sector type `+128M`, then for the partition type enter `ef00`

![Kali live gdisk new efi partition](/images/installing-kali-linux-on-hyperv/kali-live-gdisk-new-efi.png)

Type `n` and choose all the default options to create a linux partition with the remainder of the space.

Type `p` to print the partition table to verify everything is correct.

![Kali live gdisk new linux partition](/images/installing-kali-linux-on-hyperv/kali-live-gdisk-new-linux.png)

Type `w` to save the partition table and exit gdisk.

Run `mkdosfs -F 32 -n EFI /dev/sda1` to format the EFI partition.

Run `mkfs.ext4 -G 4096 /dev/sda2` to format the Kali partition. [Microsoft recommends](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/best-practices-for-running-linux-on-hyper-v) setting the number of groups to 4096.

![Kali live gdisk save partitions and format](/images/installing-kali-linux-on-hyperv/kali-live-gdisk-save-and-mkfs-ext4.png)

## Install Kali Linux
Open the VM settings and change the image file to the Kali netinstall iso that you downloaded earlier.

![VM settings select kali netinstall iso](/images/installing-kali-linux-on-hyperv/vm-settings-select-kali-netinst.png)

Run the VM and choose the default boot option.

![Kali netinstall boot menu](/images/installing-kali-linux-on-hyperv/kali-netinstall-boot-menu.png)

Set your language, location, and keyboard.

![Kali netinstall select language](/images/installing-kali-linux-on-hyperv/kali-netinstall-language.png)

![Kali netinstall select location](/images/installing-kali-linux-on-hyperv/kali-netinstall-location.png)

![Kali netinstall select keyboard](/images/installing-kali-linux-on-hyperv/kali-netinstall-keyboard.png)

Choose your hostname and domain name (I left domain blank).

![Kali netinstall set host name](/images/installing-kali-linux-on-hyperv/kali-netinstall-host-name.png)

![Kali netinstall set domain name](/images/installing-kali-linux-on-hyperv/kali-netinstall-domain-name.png)

Enter your name, user name, and password.

![Kali netinstall set full user name](/images/installing-kali-linux-on-hyperv/kali-netinstall-user-full-name.png)

![Kali netinstall set user name](/images/installing-kali-linux-on-hyperv/kali-netinstall-user-name.png)

![Kali netinstall set password](/images/installing-kali-linux-on-hyperv/kali-netinstall-password.png)

Set your time zone.

![Kali netinstall set timezone](/images/installing-kali-linux-on-hyperv/kali-netinstall-time-zone.png)

Choose manual partitioning because we already created the partition table and formatted.

![Kali netinstall manual partition](/images/installing-kali-linux-on-hyperv/kali-netinstall-manual-partition.png)

The EFI partition is already set up so we want to edit the second partition for Kali.

![Kali netinstall select partition to change](/images/installing-kali-linux-on-hyperv/kali-netinstall-partition-select.png)

Set it to ext4 partition, set the mount point as `/` and keep the existing data.

![Kali netinstall partition options](/images/installing-kali-linux-on-hyperv/kali-netinstall-partition-options.png)

Write changes to disk.

![Kali netinstall finish partitioning](/images/installing-kali-linux-on-hyperv/kali-netinstall-partition-finish.png)

Continue without a swap partition.

![Kali netinstall continue without swap](/images/installing-kali-linux-on-hyperv/kali-netinstall-partition-swap.png)

Continue with installation.

![Kali netinstall save partition settings and continue](/images/installing-kali-linux-on-hyperv/kali-netinstall-partition-continue.png)

Set up a network proxy (I left this blank.)

![Kali netinstall enter proxy settings](/images/installing-kali-linux-on-hyperv/kali-netinstall-proxy.png)

Choose the software you want installed (I went with the default options)

![Kali netinstall select software to install](/images/installing-kali-linux-on-hyperv/kali-netinstall-software-selection.png)

I had an error message after all of the packages downloaded. I hit continue a few times and it continued installing where it left off without giving another error.

![Kali netinstall software installation error](/images/installing-kali-linux-on-hyperv/kali-netinstall-error.png)

![Kali netinstall continue after error message](/images/installing-kali-linux-on-hyperv/kali-netinstall-continue-after-error.png)

Kali is installed! Shut down the machine and remove the DVD drive in the settings.

![Kali netinstall finished](/images/installing-kali-linux-on-hyperv/kali-netinstall-finished.png)

![VM settings remove DVD drive](/images/installing-kali-linux-on-hyperv/vm-settings-remove-dvd-drive.png)

## Install linux-vm-tools
Start up the Kali VM and log in.

![Kali Linux boot menu](/images/installing-kali-linux-on-hyperv/kali-boot-menu.png)

![Kali Linux login screen](/images/installing-kali-linux-on-hyperv/kali-login-screen.png)

Open a terminal and run `git clone https://github.com/markramige/linux-vm-tools` to download the install script

Now run `sudo linux-vm-tools/kali/install.sh`

This script will update Kali, install xrdp, and change some settings that are necessary for the enhanced session mode to activate. Reboot the machine when it finishes.

![Kali Linux tools install](/images/installing-kali-linux-on-hyperv/kali-linux-vm-tools-install.png)

You can now set the resolution of the RDP session. This screen will pop up every time you boot unless you save the settings. If this screen doesn’t pop up the first time you reboot, try shutting down the machine completely and starting it again. Alternatively you can save the machine state at the kali login screen and the next time you start it up it will switch to the enhanced session.

![Hyper-V enhanced session menu](/images/installing-kali-linux-on-hyperv/hyperv-enhance-resolution.png)

You should see the xrdp login screen instead of the default Kali login screen.

![Xrdp login screen](/images/installing-kali-linux-on-hyperv/xrdp-login-screen.png)

## Adjust XFCE settings
I’ve found that disabling desktop compositing makes it feel much faster.

![XFCE disable compositing](/images/installing-kali-linux-on-hyperv/xfce-disable-compositing.png)

Disable desktop blanking and locking

![XFCE disable display blanking](/images/installing-kali-linux-on-hyperv/xfce-disable-display-blanking.png)

Set the desktop background to a solid color

![XFCE desktop background set to black](/images/installing-kali-linux-on-hyperv/xfce-desktop-background.png)

All done!

![Kali desktop finished](/images/installing-kali-linux-on-hyperv/kali-finished-desktop.png)

## Future improvements
It’s possible to enable sound with [pulseaudio-module-xrdp](https://github.com/neutrinolabs/pulseaudio-module-xrdp). I might try getting that to work if I ever need sound support. Also, usb-over-ip might be a possibility, but I don’t know what types of devices are supported with that.

## Conclusion
If you need to run Windows as your host OS and aren’t willing to give up WSL 2, then Hyper-V is a decent alternative to VirtualBox and VMware Workstation. It’s not quite as polished as the other options for desktop Linux, but hopefully Microsoft makes some improvements in the future to make it run more smoothly.
