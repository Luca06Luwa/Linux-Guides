# Arch Linux UEFI Install Guide
Small note: This took soo long to rewrite this guide as there is soo much stuff in it that had to be rewritten from scratch.

This guide assumes that your default language is english.

## 1. Basic initial Setup
a. Run the command `ls /sys/firmware/efi/efivars` to check if your booted into UEFI.</br>
b. Run `timedatectl status` to ensure the date and time is accurate.

## 2. Networking
If you're not going to be using WI-FI, then skip steps b to h.

### Part One (Wi-Fi Setup).
a. Run `ip link` to identify your network setup.</br>
b. Run `iwctl` to configure your Wi-Fi connection.</br>
c. Once in iwd run `device list` to list your Wi-Fi card.</br>
d. Run `station [Your wifi card] scan` to scan the local area for networks.</br>
e. Run `station [Your wifi card] get-networks` to list the available networks.</br>
f. Run `station [your wifi card] connect "[your network]"` to connect to your network.</br>
g. Press `ctrl + d` to exit iwd</br>
h. ip link (To verify that you're getting an ip connection to Wi-Fi).

### Part Two (Testing Connection).
Run `ping archlinux.org` to test the internet connection.</br>
Note: You can press `ctrl + c` to stop pinging.

## 3. Creating the Partition Tables.
If you plan on dual booting Windows 10/11, STOP this guide is not for you.

Note: If you have a blank drive that you know is empty, then skip step c.

a. Run `lsblk` to see what hard drives you have installed in your PC.</br>
b. If you cannot identify what drive(s) you have installed, run `hdparm -i /dev/the_disk_to_be_partitioned` to double check that you've selected the right drive.</br>
c. If you only have one drive woth another OS on it and want to install clean, run `gdisk /dev/the_disk_to_be_partitioned`.
- Press `x` to enable expert mode.
- Press `z` to delete the entire content of the drive.

d. Run `cgdisk /dev/the_disk_to_be_partitioned` to format the drive.</br>
e. Format the drive like this.
| Partition | Minimum Allocation | Maximum Allocation | Partition Type | Partition Name |
| --------- | ------------------ | ------------------ | -------------- | -------------- |
| Boot partition | Default | 1024MiB | EF00 | boot |
| Swap partition | Default | 16GiB | 8200 | swap |
| Root partition | Default | 32GiB | Default | root |
| Home partition | Default | The Remainder of the drive | Default | home |

Help: EF00 = uefi bootable partition, 8200 = swap and 8300 = linux filesystem.

f. write changes to disk

## 4. Format and Mount your partitions.
a. Run `lsblk` to see what your doing.</br>
b. Run `mkswap /dev/swap_partition` to format the swap partition and run `swapon /dev/swap_partition` to enable swap.</br>
c. Run `mkfs.ext4 /dev/root_partition` to format the root partition and run `mount /dev/root_partition /mnt` to mount the drive.</br>
d. Run `mkfs.ext4 /dev/home_partition` to format the home partition and run `mount --mkdir /dev/home_partition /mnt/home` to create and mount the home partition </br>
e. Run `mkfs.fat -F 32 /dev/efi_system_partition` to format the boot partition.

### Mount Boot loader Partitions.
Here is a little choose your own adventure bit for this install guide. There are two commonly boot loaders that people tend to install. Choose the one you prefer and forget about the other one.

6a. Systemd-Boot. (Hard Mode)</br>
mount --mkdir /dev/efi_system_partition /mnt/boot

6b. Grub Boot loader. (Easy Mode)</br>
As of grub version v2.06 r403 the issue where grub would brick some installs is still prominent so for right now DO NOT USE GRUB IF YOU DO NOT WANT A BROKEN INSTALL!!!!</br>

mount --mkdir /dev/efi_system_partition /mnt/boot/efi
