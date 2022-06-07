# roger-skyline-1

Our task with roger-skyline is to install a VM with the Linux OS of our choice. We have to have a VM disk size of 8GB, one partition has to be 4.2GB, and we also need to setup some networking and security for our VM.

The second part of roger-skyline is to install a web server on the VM.

## How to install Artix Linux with roger-skyline-1 project requirements

### Virtualization Stack

My preferred virtualization stack is QEMU+Virt-Manager. The reason people prefer QEMU over VirtualBox is its performance. Virt-Manager is a front-end for QEMU because QEMU by itself is a terminal application.

How to install QEMU+Virt-Manager: https://youtu.be/wxxP39cNJOs

### Install

Downloade the ISO file from here: https://artixlinux.org/download.php

Just select the base verison of runit and then run the ISO on your hypervisor.

When setting up the VM remember to make the disk image size 8GB.

Login with `root` as username and `artix` as password.

#### Partitioning

Use the `cfdisk` program to manage and display a disk partition table.

Open `cfdisk /dev/vda` 

*For me it's vda but for some it might be /dev/sda or if you're not doing this to a VM it might be something beginning with nvme.*

  1. Select gpt as label type.

*We need to have two partitions. The root partition and the home partition. If you have a UEFI system (not a VM)  you need the boot partition, and if you are limited in memory, you probably want swap space. The swap space is for when you have limited memory and the operating system can use your hard drive as RAM.*

  2. Select free space and make a new partition of 4.2G. Then a second one with 3.8G. The bigger one is for the home.

  3. Write the changes and quit the program.

![](pic-selected-220607-1359-21.png)

If you use `lsblk` it shoud look something like that above.

#### Creating file systems

We will create a ext4 file system with the `mkfs` program.

Make sure to make the home partition the bigger one.

`mkfs.ext4 -L HOME /dev/vda1`

`mkfs.ext4 -L ROOT /dev/vda2`

We are labeling them so mounting is easier.

#### Mounting

After creating the file systems we need to mount the partition so we can chroot to them.

`mount /dev/disk/by-label/ROOT /mnt`

`mkdir /mnt/home`

`mount /dev/disk/by-label/HOME /mnt/home`

#### The Actual Install

Run the following:

`basestrap /mnt base runit elogind-runit linux nano` <-- Install the base system and kernel

`fstabgen -U /mnt >> /mnt/etc/fstab` <-- For defining how disk partitions are mounted

#### Configure The System

Now we have to chroot to get inside the system.

Run `artix-chroot /mnt`

After this step we are going to be inside our system.

Here you can change to using bash by just typing `bash`.

#### Systemclock

Setup systemclock with `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`,

and `hwclock --systohc`

#### Localization

Edit and uncomment your locales you want.

`nano /etc/locale.gen`

Then generare locale using `locale-gen`

#### Boot Loader
This is perhaps the most important step. We are going to use grub for our boot loader.

`pacman -S grub os-prober efibootmgr` <-- Install grub

`grub-install --recheck /dev/sda` <-- Add `--force` if it's complaining about blacklists

`grub-mkconfig -o /boot/grub/grub.cfg`

