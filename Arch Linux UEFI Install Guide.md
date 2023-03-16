# Arch Linux UEFI Install Guide
Small note: This took soo long to rewrite this guide as there is soo much stuff in it that had to be rewritten from scratch.

This guide assumes that your default language is english and you are on desktop.

## 1. Basic initial Setup
a. Run the command `ls /sys/firmware/efi/efivars` to check if your booted into UEFI.<br>
b. Run `timedatectl status` to ensure the date and time is accurate.

## 2. Networking
If you're not going to be using WI-FI, then skip steps b to h.

### Part One (Wi-Fi Setup).
a. Run `ip link` to identify your network setup.<br>
b. Run `iwctl` to configure your Wi-Fi connection.<br>
c. Once in iwd run `device list` to list your Wi-Fi card.<br>
d. Run `station [Your wifi card] scan` to scan the local area for networks.<br>
e. Run `station [Your wifi card] get-networks` to list the available networks.<br>
f. Run `station [your wifi card] connect "[your network]"` to connect to your network.<br>
g. Press `ctrl + d` to exit iwd<br>
h. Run `ip link` again to verify that you're getting an ip connection to Wi-Fi.

### Part Two (Testing Connection).
Run `ping archlinux.org` to test the internet connection.<br>
Note: You can press `ctrl + c` to stop pinging.

## 3. Creating the Partition Tables.
If you plan on dual booting Windows 10/11, STOP this guide is not for you.

Note: If you have a blank drive that you know is empty, then skip step c.

a. Run `lsblk` to see what hard drives you have installed in your PC.<br>
b. If you cannot identify what drive(s) you have installed, run `hdparm -i /dev/the_disk_to_be_partitioned` to double check that you've selected the right drive.<br>
c. If you only have one drive woth another OS on it and want to install clean, run `gdisk /dev/the_disk_to_be_partitioned`.
- Press `x` to enable expert mode.
- Press `z` to delete the entire content of the drive.

d. Run `cgdisk /dev/the_disk_to_be_partitioned` to format the drive.<br>
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
a. Run `lsblk` to see what your doing.<br>
b. Run `mkswap /dev/swap_partition` to format the swap partition and run `swapon /dev/swap_partition` to enable swap.<br>
c. Run `mkfs.ext4 /dev/root_partition` to format the root partition and run `mount /dev/root_partition /mnt` to mount the drive.<br>
d. Run `mkfs.ext4 /dev/home_partition` to format the home partition and run `mount --mkdir /dev/home_partition /mnt/home` to create and mount the home partition <br>
e. Run `mkfs.fat -F 32 /dev/efi_system_partition` to format the boot partition.

### Mount Bootloader Partitions.
Here is a little choose your own adventure bit for this install guide. There are two commonly boot loaders that people tend to install. Choose the one you prefer and forget about the other one.

Systemd-Boot. (Hard Mode)<br>
This bootloader is what I would recommended you as the packages are preinstalled onto system during install.

GRand Unified Bootloader / GRUB. (Easy Mode)<br>
This bootloader is the most common across most distro's and has the most documentation.<br>
As of release version v2.06 r415 the issue where GRUB would brick some installs is still prominent so for right now:<br>
PLEASE REGENERATE THE GRUB CONFIG EVERY TIME YOU UPDATE GRUB TO AVOID THE RISK OF BRICKS!!!!

| Bootloader | Instructions |
| ---------- | ------------ |
| Systemd-Boot | Run `mount --mkdir /dev/efi_system_partition /mnt/boot` to create and mount the boot partition for Systemd-Boot. |
| GRUB | Run `mount --mkdir /dev/efi_system_partition /mnt/boot/efi` to create and mount the boot partition for GRUB. |

## 5. Configure Mirrorlist.
This step isn't really necessary but I would highly recommend it as it sorts the servers from best to worst.

Note: Since we don't have a GUI interface for file management we must do everything through command line.

a. Run `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup` to create a backup.<br>
b. Run `nano /etc/pacman.d/mirrorlist` to see if all the servers that are listed in the file are uncommented.<br>
c. Once exited nano, run `pacman -Sy` to update the package database on the ISO.<br>
d. Run `pacman -S pacman-contrib` to install the tools needed for sorting the servers.<br>
e. Run `rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist` to sort the servers in the backup file and copy it to the main file.

## 6. Download/Installing Essential Packages.
This is where your going to actually install your system.

The following packages that it will install are the necessary core functions and the drivers for some wifi cards.

Run `pacstrap -K /mnt base base-devel linux linux-headers linux-firmware linux-firmware-marvell linux-firmware-whence nano` to install the packages.

## 7. Generating the fstab and chrooting into the install.
This step is important as without doing this the system wont know what it's doing.

a. Run `genfstab -U /mnt >> /mnt/etc/fstab` to generate the fstab file.<br>
b. Run `arch-chroot /mnt` to gain access to your install.

## 8. Localisation and Timezone.
This step is to tell Arch Linux where you are and to set the system clock.

WARNING: Anything and everything in this part is important. If you mess up when entering these commands your install is dead.

a. Run `nano /etc/locale.gen` and scroll down to your locale of you know it. If you don't then uncomment `en_US.UTF-8 UTF-8`.<br>
b. Once uncommented your locale of choice run `locale-gen` to generate the locales.<br>
c. Even though you've already assigned the locale, you still need to echo the locale for older programs. To do this run `echo "LANG=[the locale you selected].UTF-8" >> /etc/locale.conf` to set the locale.<br>
d. This step is important and should be done either way. Run `export LANG=[the locale you selected].UTF-8`.<br>
e. Skip this step if you have a us keyboard layout. If you have a keyboard other than us run `echo "KEYMAP=us" >> /etc/vconsole.conf`.<br>
f. Run `ln /usr/share/zoneinfo` to list the unix timezones.<br>
g. Once found your timesone run `ln -sf /usr/share/zoneinfo/[Your Country Here]/[Your Timesone Here] /etc/localtime` to add a symbolic link for your time.<br>
h. Run `hwclock --systohc --utc` to set the hardware clock.

## 11. Configure Pacman/Package Manager.
This step is where you will configure pacman to be able to download faster and enable the Multilib repo for 32-bit programs.

a. Run `nano /etc/pacman.conf` to enter the pacman config file.<br>
b. Uncomment the line that you see below.<br>

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

c. In the Misc Options area, add/uncomment the following items. `ParallelDownloads = 5`, `Color` and `ILoveCandy`.<br>
d. Once saved run `pacman -Sy` to update and download the repos.

## 12. Installing more packages and enabling system services.
This part is where you're going to install some more packages and some drivers for basic networking and enable some system functions.

a. Run `pacman -S git networkmanager reflector pacman-contrib bluez bluez-utils bash-completion` to install the packages.<br>
b. To make sure your CPU has no active exploits on it firmware, you need to install the microcode. To install it run `pacman -S [Your CPU Brand]-ucode`.<br>
c. Enable the following services to start the drivers.
```
systemctl enable bluetooth.service
systemctl enable NetworkManager.service
systemctl enable fstrim.timer
systemctl enable reflector.timer
```

## 14. Hostname Configuration and User Setup.
This step is where you'll name the computer and add your user account. 

a. Run `echo "[Insert Computer Name Here]" >> /etc/hostname` to set the name for the computer.<br>
b. Run `nano /etc/hosts` and add the following into the file.
```
127.0.0.1        localhost
::1              localhost
127.0.1.1        [Add same hostname as before. lol]
```

c. Run `passwd` to set the root password.<br>
d. Run `useradd -m -G wheel,storage,power -s /bin/bash [Insert Username Here]` to create your user account.<br>
e. Run `passwd [Insert Username Here]` to set the password for the user account you just created.<br>
f. Run `EDITOR=nano visudo` to and edit the following permissions.<br>
Uncomment `%wheel ALL=(ALL) ALL` and add `Defaults rootpw` to the bottom of the file.

## Bootloader.
This step will be based off of which bootloader you decided to partition your hard drive for.

If you chose to install GRUB then ONLY do 15a.<br>
If you chose to install Systemd-Boot then ONLY do 15b.

### 15a. GRUB. (Linux dual boot/Easy Mode)
a. Run `pacman -S grub efibootmgr os-prober` to install the necessary packages.<br>
b. Run `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB` to inject and install GRUB to your system.<br>
c. Run `grub-mkconfig -o /boot/grub/grub.cfg` to generate the configuration files.

### 15b. Systemd-Boot. (Requires manual entries/Hard Mode)
a. Re-run `ls /sys/firmware/efi/efivars` to check if the efi firmware is mounted and installed.<br>
b. Run `bootctl install` to inject and install Systemd-Boot to your system.<br>
c. Run `systemctl enable systemd-boot-update.service` to enable the updater script.<br>
d. Run `nano /boot/loader/entries/arch.conf` and add the following lines.
```
title Arch Linux
linux /vmlinuz-linux (change this depending on what kernel you have).
initrd /initramfs-linux.img
initrd /intel-ucode.img or /amd-ucode.img
```

e. Once added everything into the file, run `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition) rw" >> /boot/loader/entries/arch.conf` to add the partition id of the root partition so that it tells Arch Linux that it will boot to that drive only.


## 16. Graphics Drivers
This step is necessary if you want your GPU to be working properly. The drivers are sorted based on what manufacturer your card is from so select the one that matches your card.

| Manufacturer | Instructions |
| ------------ | ------------ |
| AMD | Run `pacman -S xf86-video-amdgpu libva-mesa-driver mesa opencl-mesa vulkan-radeon lib32-libva-mesa-driver lib32-mesa lib32-opencl-mesa lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for AMD cards. |
| INTEL | Run `pacman -S xf86-video-intel mesa intel-compute-runtime intel-media-driver vulkan-intel lib32-mesa lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for INTEL cards. |
| NVIDIA (PROPRIETARY) | Run `pacman -S nvidia-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for MAXWELL series cards or newer. |
| NVIDIA (Open GPU Kernel Modules) | Run `pacman -S nvidia-open-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for TURING series cards or newer. |


## Configure Drivers for KMS/Wayland Support.
This part should only be done with the version that matches your card.

WARNING: If you for whatever reason mess up on this your system is bricked!

### 17a. NVIDIA
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)`<br>
b. Once added those modules, run `mkinitcpio -P` to regenerate the initramfs.<br>
c. This step will vary depending on your bootloader so make sure you select the correct one. DRM kernel mode setting.

| Bootloader | Instructions |
| ---------- | ------------ |
| GRUB | 1. Run `nano /etc/default/grub` and modify the `GRUB_CMDLINE_LINUX_DEFAULT=` line to look like this. `GRUB_CMDLINE_LINUX_DEFAULT=... nvidia-drm.modeset=1`.<br>2.Once added, run `grub-mkconfig -o /boot/grub/grub.conf` to regenerate the grub configuration files. |
| Systemd-Boot | Run `nano /boot/loader/entries/arch.conf` and at the end of the options line add `nvidia-drm.modeset=1`. |

d. In order for the initramfs to update with the drivers for your NVIDIA card, we must create a hook. TO do this run `mkdir /etc/pacman.d/hooks` to create the directory for the hook files.<br>
e. Run `nano /etc/pacman.d/hooks/nvidia.hook` to create the file and edit the file to look like this.
```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia-dkms or nvidia-open-dkms

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
```

### 17b. AMDGPU
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... amdgpu ...)`<br>
b. Once added those modules, run `mkinitcpio -P` to regenerate the initramfs.


### 17c. INTEL
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... i915 ...)`<br>
b. Once added those modules, run `mkinitcpio -P` to regenerate the initramfs.

## 18. Unmount drives and Reboot system.
Congratulations. You have sucessfully installed the basic form of Arch Linux. However you're not done just yet.
a. Type `exit` to return back to the install drive.<br>
b. Type `umount -r /mnt` to safely unount the partitions.<br>
c. `reboot`
