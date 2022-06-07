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

We will create a ext4 file system with the `mkfs.ext4` command in the terminal.

Make sure to make the home partition the bigger one.

`mkfs.ext4 -L HOME /dev/vda1`

`mkfs.ext4 -L ROOT /dev/vda2`

#### Mounting

After creating the file systems we need to mount the partition so we can chroot to them.

`mount /dev/disk/by-label/ROOT /mnt`

`mkdir /mnt/home`

`mount /dev/disk/by-label/HOME /mnt/home`


