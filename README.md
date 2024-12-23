# Arch Linux Guide

A simple guide to help you install arch seamlessly
## Table of Contents

- [Requirements](#Requirements)
- [Installation Media Creation](#Installation-Media-Creation)
- [Booting with Installation Media](#Booting-with-Installation-Media)
- [Network Configuration](#Network-Configuration)
- [Partitioning](#Partitioning)
- [Installing Base System](#Installation-Base-System)
- [Configuring the Fresh Installation](#Configuring-the-Fresh-Installation)
- [Booting into the new Installation](#Booting-into-the-new-Installation)

## Requirements

- A USB Pendrive with a capacity of at least 2GB
- A Laptop or a PC which supports `Legacy Bios`

## Installation Media Creation
<p> Download the Arch Linux ISO from the following URL: <a href="https://archlinux.org/download/" href="_blank">Arch ISO</a> </p>
<p>Meanwhile,
Install the program to install the ISO to the USB pendrive for your <br>
OS which you will use to make the installation media: <a href="https://etcher.balena.io/#download-etcher" href="_blank">Media Creation Tool</a>
</p>

Now open the Media Creation Tool `balena etcher` which you just installed.

<img width="520" alt="image" src="https://github.com/rohansharma-developer/arch-linux-guide/assets/107614947/ba951fae-c7c1-4cdd-a22c-14991fca98eb">

 - Click on `Flash from File` and choose the Arch Linux ISO file which you just downloaded

<img width="520" alt="image" src="https://github.com/rohansharma-developer/arch-linux-guide/assets/107614947/b1b6ac18-c8e6-48c2-b262-e0a5035813bb">

> [!CAUTION]
> This process will wipe out all your data on the pendrive. Please backup your data before proceeding

 - Now, Plug in your USB pendrive in your PC and click on `Select Target` and select your Pendrive from the list

Now begin the Media Creation Process by clicking on the `Flash` Button. Let the process run until the application signals the process as successfully completed.

Great! You have created your Arch Linux Installation Media

## Booting with Installation Media

 - Shutdown your PC from your current OS
 - Connect your USB Pendrive to your Computer
 - Turn on your computer and Boot into your Computer's BIOS and enable USB Boot
 - Save the BIOS settings
 - The computer will automatically boot from the pendrive

## Network Configuration

 - Once the OS fully boots up begin with the below process

 - Start with Entering in the Network Config Utility, Type `iwctl` and press ENTER
   ``` bash
   iwctl
   ```
 - Enter `station wlan0 connect <wifi name>` to connenct to your Wi-fi router
   ``` bash
   station wlan0 connect <wifi name>
   ```
 - Enter `station wlan0 show status` to verify the computer's connection to the router
   ``` bash
   station wlan0 show status
   ```
 - After succesfully connecting to your router, Simply exit the Utility by typing `exit`
   ``` bash
   exit
   ```
<center>

**This completes the network configuration part of the installation process!**

</center>

## Partitioning

 - Now we need to create three partition on a hard disk. For reference, here is a table below:

| ID | Size   | Filesystem | Directory |
| -- | ------ | ---------- | --------  |
| 1  | 100MB  |   FAT32    | /boot/efi |
| 2  | 4-16GB |   SWAP     | N/A       |
| 3  | Custom |   EXT4     | /         |

 - Enter `lsblk` to check the hard drive which you want to partition
   
 - Enter `cfdisk /dev/sdX` to Open the disk partition utility
   
 - Replace the X with the letter of the drive which you want to partition

If The utility asks you to select a label type, <br> 
choose `gpt` from the options by navigating with the arrow keys

- For **Partition 1** Select the new option for creating a partition

- Enter the disk size as `100M`

- For **Partition 2**, Again select the `new` option

- Partition 2 will be used for virtual memory, the Recommended Size is 8GB but you can maximise it to 16GB RAM

- If your preferred disk size is in Gigibyte size, use the keyword `G` such as `8G` for specifying the partition size.

- For **Partition 3**, Selecting the same option...

- This partition is where you will keep all your data and also where linux will be installed

- Partition size of **8GB** is minimum, but **32GB AND ABOVE** is recommended for daily usage

#### Formating the partitions

- Start with formatting the partition tables according to the table above

- Format the first partition from the table above with the following and the replace the 'X' with the installation disk and the parition number:

``` bash 
mkfs.fat -F 32 /dev/sdX
```

- Format the second partition to **EXT4** Format

``` bash 
  mkfs.ext4 /dev/sdX
```

- Format the third partition to initialise the swap

``` bash 
mkswap /dev/sdX
```

## Mount the Partitions

- Mount the **Partition 2 [Main Partition][EXT4]** to `/mnt`

``` bash
mount /dev/sdX /mnt
```

- Mount the **Partition 1 [EFI Partition][FAT32]** to `/mnt/boot`

``` bash
mount /dev/sdX /mnt/boot
```

- Mount the **Partition 3 [Swap Partition][Linux Swap]**

``` bash
swapon /dev/sdaX
```


## Installing Base System
``` bash 
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr nano networkmanager
```

## Configuring the Fresh Installation

- Generate an fstab file (use -U or -L to define by UUID or labels, respectively)

``` bash
genfstab -U /mnt >> /mnt/etc/fstab
```

- Change user to new installation

``` bash
arch-chroot /mnt
```

- Set the time zone for the newly created installation

``` bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

- Synchronise the changed timezone settings

``` bash
hwclock --systohc
```
- Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed UTF-8 locales:

``` bash
nano /etc/locale.gen
```

- After editing, close the editor by pressing `CTRL+X`, then press `y` and then press `ENTER`

- Generate the locales by running:

``` bash
locale-gen
```

- Create the hostname file,  and then give it any name:

``` bash
nano /etc/hostname
```

- After editing, close the editor by pressing `CTRL+X`, then press `y` and then press `ENTER`

- Set a password for the `SUDO` user by the following command:

``` bash
passwd
```

- Create your user account on the installation (Replace the `username` with your own)

``` bash
useradd -m -G wheel -s /bin/bash username
```

- Set the password for the newly created user account (Replace the `username` with your account username)

``` bash
passwd username
```

- Edit the sudoers file to provide your newly created user SUDO or Admin Privelages

``` bash
EDITOR=nano visudo
```

- Uncomment the line saying `wheel ALL(ALL) ALL` and then exit the editor

- Change terminal user to the newly created one:

``` bash
su username
```

- Update the package database (the command will request the user to enter the password for the current account before proceeding)

``` bash
sudo pacman -Syu
```

## Enabling Services

- Enable NetworkManager (responsible for wifi management)

``` bash
systemctl enable NetworkManager
```

## Installing BootLoader

- Install the `GRUB` bootloader to the disk where linux is installed (don't specify any partition number)

``` bash
grub-install /dev/sdaX
```

- Generate the GRUB configuration file:

``` bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## Booting into the new Installation

- Detach the chroot interface by executing exit two times

``` bash
exit
exit
```

- Unmount all mounted drives and partitions

``` bash
umount -a
```

- Finally, reboot your pc:

``` bash
reboot
```
