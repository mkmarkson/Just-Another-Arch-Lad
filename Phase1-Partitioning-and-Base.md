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

For partitioningIn I use `fdisk`  
To check your disks use `lsblk`  
Since all of my computers drives are M.2, I will be partitioning nvme0n1.  
But if you are using regular hard disk, it might be sda,sdb or what not.  
Just try to make sure you are not partitioning your usb stick :)  

`fdisk /dev/nvme0n1`  
`g`	#Since I have UEFI system I will make GTP label  
`n`  
Press enter for default partition number  
Press enter for default first sector  
`+500M`	#For last sector this will make 500MB boot partition

(OPTIONAL) If you want to use systemd-boot bootloader  
`t`	#For partition type  
`1`	#For EFI System

Root Partition  
`n`  
Press enter for default partition number(2)  
Press enter for default first sector  
`+35G`	#35G for root partition is a lot, most people only need 20 or 25

Home partition  
`n`  
Press enter for default partition number(3)  
Press enter for default first sector  
Press enter for default last sector	#Gives reminder of the disk to home partition

Write the changes  
`w`


