# Partitioning and the base

## Prerequisite

First check that you have ip or you are connected to the internet

`ip a`   
`ping 1.1.1.1`

Incase you are using laptop setup wifi and test internet connection

`wifi-menu`

We neet to sync network time protocol

`timedatectl set-ntp true`

## Partitioning

For partitioning I use `fdisk`  
To check your disks use `lsblk`  
Since all of my computers drives are M.2, I will be partitioning nvme0n1.  
But if you are using regular hard disk, it might be sda,sdb or what not.  
Just try to make sure you are not partitioning your usb stick :)  

---

`fdisk /dev/nvme0n1`  
`g`	#Since I have UEFI system I will make GTP label  
`n`  
Press enter for default partition number  
Press enter for default first sector  
`+500M`	#For last sector this will make 500MB boot partition

---

(OPTIONAL) If you want to use systemd-boot bootloader  
`t`	#For partition type  
`1`	#For EFI System

---

Root Partition  
`n`  
Press enter for default partition number(2)  
Press enter for default first sector  
`+35G`	#35G for root partition is a lot, most people only need 20 or 25

---

Home partition  
`n`  
Press enter for default partition number(3)  
Press enter for default first sector  
Press enter for default last sector	#Gives reminder of the disk to home partition

---

Write the changes  
`w`

## Format filesystems

`mkfs.fat -F32 /dev/nvme0n1p1`  
`mkfs.ext4 /dev/nvme0n1p2`  
`mkfs.ext4 /dev/nvme0n1p3`  

## Mounting

`mount /dev/nvme0n1p2 /mnt` #First mount root partition  
`mkdir /mnt/boot`  
`mkdir /mnt/home`  
`mount /dev/nvme0n1p1 /mnt/boot`  
`mount /dev/nvme0n1p3 /mnt/home`  

## Install system base

`pacstrap /mnt base linux linux-firmware vim`	#vim is my editor of choice for later. Use nano or what not

## Generate fstab file

`genfstab -U /mnt >> /mnt/etc/fstab`  
`cat /mnt/etc/fstab`	#Check that 3 lines about partition exist

## Enter into the installation

`arch-chroot /mnt`

## Swap

`dd if=/dev/zero of=/swapfile bs=1M count=8192 status=progress`	#8GB swapfile  
`chmod 600 /swapfile`  
`mkswap /swapfile`  
`swapon /swapfile`  

Add swapfile into fstab file  
Add line to the bottom of the file  
`/swapfile none swap defaults 0 0`  

## Localisation

`ln -sf /usr/share/zoneinfo/Europe/Tallinn /etc/localtime`  
`hwclock --systohc`  

Now remove comment in /etc/locale.gen  
`en_US.UTF-8 UTF-8`  
`echo "LANG=en_us.UTF-8" >> /etc/locale.conf`  

## Hostname

`echo "arch" >> /etc/hostname`  

Put this in /etc/hosts  
`
127.0.0.1 localhost
::1	  localhost
127.0.1.1 arch.localdomain arch

## Root password

`passwd`

## Additional packages

`pacman -S efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog os-prober mtools dosfstools base-devel linux-headers`

## Install systemd-boot

`bootctl --path=/boot install`  
`cd /boot/loader`  
  
Edit loader.conf  
1.Uncomment first line  
2.Last line to `default arch-\*`
  
`cd entries/`  
`vim arch.conf`  
  
Add the following:  
`title Arch Linux`  
`linux /vmlinuz-linux`  
`initrd	/initramfs-linux.img`  
`options root=/dev/nvme0n1p2 rw`

## Enable network manager

`systemctl enable NetworkManager`

## Add new user

`useradd -mG wheel username`  
`passwd username`  
`EDITOR=VIM visudo`  
Remove comment from the first `%wheel ALL=(ALL) ALL  

## Finish installation

`exit`  
`umount -a`  
`reboot`


## Post installation packages

Install graphics based on your hardware  
`sudo pacman -S xf86-video-qxl`  
`sudo pacman -S xf86-video-intel`(or amdgpu)  
`sudo pacman -S nvidia nvidia-utils`  

