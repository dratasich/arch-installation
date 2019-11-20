Installation Logs
=================

* Features:
  * `/home` is a separate partition
* Applied on: Virtual Box VM
* Instructions followed (in the order of application):
  * https://wiki.archlinux.org/index.php/Installation_guide
  * https://wiki.archlinux.org/index.php/Securely_wipe_disk
  * https://wiki.archlinux.org/index.php/Partitioning#Choosing_between_GPT_and_MBR
  * https://wiki.archlinux.org/index.php/Fdisk#Create_a_partition_table_and_partitions
  * https://wiki.archlinux.org/index.php/Swap
  * https://wiki.archlinux.org/index.php/Installation_guide
  * https://wiki.archlinux.org/index.php/EFI_system_partition#Format_the_partition
  * https://www.archlinux.org/mirrors/status/
  * https://wiki.archlinux.org/index.php/GRUB#Installation_2
  * https://wiki.archlinux.org/index.php/GRUB#Generated_grub.cfg

Prerequisites:
* UEFI must be enabled on boot

```bash
$ loadkeys de-latin1

# UEFI is enabled? (folder exists)
$ ls /sys/firmware/efi/efivars

$ ip link
$ ping archlinux.org

$ timedatectl set-ntp true
$ timedatectl status

# check disks
$ fdisk -l

# wipe disk -- prepare for system encryption
# (works also for a partition, e.g., sda3)
$ dd if=/dev/urandom of=/dev/sda bs=512 status=progress

# create partition table
$ fdisk /dev/sda
g  # create GPT partition table
# EFI partition
n  # new partion
1  # number
   # first sector (default at beginning)
+512M  # size of the partition
t  # change partition type
1  # EFI type
p  # show partition table
# root partition
n
2
(default)
+2G  # 1G was not enough
t
2
24  # Linux root (x86-64)
# home partition
n
3
(default)
-512M
t
3
28  # Linux home
# swap partition
n
4
(default)
(default)
t
4
19  # Linux swap
# write/save and quit
w

# format the partitions
$ mkfs.fat -F32 /dev/sda1
$ mkfs.ext4 /dev/sda2
$ mkfs.ext4 /dev/sda3
$ mkswap /dev/sda4
$ swapon /dev/sda4

# mount partitions
$ mount /dev/sda2 /mnt
$ mkdir /mnt/efi
$ mount /dev/sda1 /mnt/efi
$ mkdir /mnt/home
$ mount /dev/sda3 /mnt/home

# edited mirror list according to https://www.archlinux.org/mirrors/status/
$ nano /etc/pacman.d/mirrorlist

# install essential packages (throws errors if partition is too small)
$ pacstrap /mnt base linux linux-firmware
$ pacstrap /mnt man-db man-pages
$ pacstrap /mnt vim
$ pacstrap /mnt grub efibootmgr

$ genfstab -U /mnt >> /mnt/etc/fstab

# change root to installed linux system
$ arch-chroot /mnt

$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc

$ vim /etc/locale.gen
# uncomment de_AT.UTF-8 and en_US.UTF-8
$ locale-gen
$ vim /etc/locale.conf
LANG=de_AT.UTF-8
$ vim /etc/vconsole.conf
KEYMAP=de-latin1

$ echo "archie" > /etc/hostname
$ vim /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   archie.localdomain archie

$ passwd
***
***

# install grub
$ grub-install --target=x86_64-efi --efi-directory=/mnt/efi --bootloader-id=GRUB
$ grub-mkconfig -o /boot/grub/grub.cfg

# finish
$ exit
$ umount -R /mnt
$ reboot
```
