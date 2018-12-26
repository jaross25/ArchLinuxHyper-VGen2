# Arch Linux on Hyper-V Generation 2 with UEFI

## 1 Introduction

I created this guide on how to install [Arch Linux](https://www.archlinux.org/) on a Hyper-V Generation 2 virtual machine that contains the UEFI-based firmware. I used an ISO image that contained the Arch Linux 2018.12.01 release which includes Kernel version 4.19.4.

## 2 Prerequisites

**Create Virtual Machine**

The specifications I used for the virtual machine are:

Memory: 8192 MB

Virtual Hard Disk: 250 GB

Number of Virtual Processors: 4

**Turn off secure boot**

If you do not turn off secure boot you will get a Hyper-V error saying:

1. SCSI DVD (0,1) Image vertification failed.

To turn off secure boot open Windows PowerShell (as an Administrator) and run:

`Set-VMfirmware -enablesecureboot off -VMName "Arch UEFI"`

(Replace "Arch UEFI" with the VM name you used)

## 3 Attach Arch Linux ISO to Virtual Machine

Head over to https://www.archlinux.org and download the latest ISO image of Arch Linux.

Go into the VM settings and make sure that the Arch Linux ISO is attached to the DVD Drive.
 
## 4 Partition

You will want to parition the HDD / SSD to turn the storage device into multiple volumes. To do this we use the `parted` utility which is a tool to manage disk partitions in linux. Below are the commands that I used:

Use `lsblk` or `fdisk -l` to view your disks on the system. In the below example I am going to use `lsblk`:

`lsblk`
 
`parted /dev/sda mklabel gpt`

`parted /dev/sda mkpart ESP fat32 1MiB 513MiB set 1 boot on`

`parted /dev/sda mkpart primary ext4 513MiB 25GiB`

`parted /dev/sda mkpart primary linux-swap 25GiB 33GiB`

`parted /dev/sda mkpart primary ext4 33GiB 100%`

Use `lsblk` or `fdisk -l` to view the partitions you just created. In the below example I am g oing to use `lsblk`:

`lsblk`

## 5 Format

Next we will want to format the partitions we created on the virtual disk and to do this we will use the `mkfs` package to build our Linux file system.
 
`mkfs.fat -F32 /dev/sda1`

`mkfs.ext4 /dev/sda2`

`mkswap /dev/sda3`

`swapon /dev/sda3`

`mkfs.ext4 /dev/sda4`
 
## 6 Mount
 
`mount /dev/sda2 /mnt`

`mkdir -p /mnt/boot/efi`

`mount /dev/sda1 /mnt/boot/efi`

`mkdir -p /mnt/home`

`mount /dev/sda4 /mnt/home`

## 7 Pacstrap

Next we are going to install Arch Linux using `pacstrap` which is the package manager in Arch:
  
`pacstrap -i /mnt base base-devel`
  
## 8 Configure fstab

Next we need to configure the fstab file which contains the data to automate the mounting of partitions.
  
`genfstab -U -p /mnt >> /mnt/etc/fstab`
 
## 9 Chroot & Set Root Password
 
`arch-chroot /mnt /bin/bash`

Set password for the root account:

`passwd`

(You will not be able to see the password characters)
 
## 10 Language &  Locale

Next we need to generate locales on the system and we do this by using `locale-gen`
 
`echo en_US.UTF UTF-8 > locale.gen`

`locale-gen`

`echo LANG=en_US.UTF-8 > /etc/locale.conf`
 
## 11 Time Zone
 
`ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime`

`hwclock --systohc --utc`

`vi /etc/hostname
   <enterhostname>`
 
## 12 Bootloader -UEFI

The bootloader I use is called GRUB and this is used to tell the system to load the Linux Kernel when the system turns on. We will use `packman` which is Arch's package manager to download grub and efibootmgr. We will then install grub using `grub-install`.
 
`pacman -S grub efibootmgr`
 
`grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck`
 
`mkdir -p /boot/efi/EFI/boot cp /boot/efi/EFI/arch_grub/grubx64.efi /boot/ef
  i/EFI/boot/bootx64.efi`
  
`grub-mkconfig -o /boot/grub/grub.cfg`
 
## 13 Few last final things
  
`exit`

`umount /mnt/home`

`umount /mnt`

`reboot`

Make sure to remove the ISO image
