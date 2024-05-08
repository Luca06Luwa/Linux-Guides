# Arch Linux UEFI Install Guide

This guide assumes that your default language is english with a us style keyboard. This guide also assumes that you are on desktop with an AMDGPU or INTEL ARC.

"Nvidia, fuck you" - Linus Torvalds

## 0. Getting the ISO
a. Go to [archlinux.org](https://archlinux.org) and click on download.<br>
b. Download the image from the torrent.<br>
c. Get some kind of [ISO burner](https://etcher.balena.io/) for a usb and just write the ISO to the USB.<br>
Note: If you do not have [a torrent client](https://www.qbittorrent.org/), you can use the download mirrors instead. Just be sure to grab the ISO file with the date printed on it.


## 1. Basic initial Setup
a. Run the command `cat /sys/firmware/efi/fw_platform_size` to check if your booted into UEFI.<br>
b. Run `timedatectl` to ensure the date and time is accurate.


## 2. Networking
If you are using ethernet instead of WI-FI, then skip part 1.

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
If you plan on dual booting Windows 10/11, STOP this guide is not for you. If you still want to dualboot with windows, figure it out yourself, I will not help you.

Note: If you have a blank drive that you know is empty, then you can skip step c.

a. Run `lsblk` to see what hard drives you have installed in your PC.<br>
b. If you cannot identify what drive(s) you have installed, run `hdparm -i /dev/the_disk_to_be_partitioned` to double check that you've selected the right drive.<br>
c. If you only have one drive woth another OS on it and want to install clean, run `gdisk /dev/the_disk_to_be_partitioned`.
- Press `x` to enable expert mode.
- Press `z` to delete the entire contents of the drive

d. Run `cgdisk /dev/the_disk_to_be_partitioned` to format the drive.<br>
e. Format the drive like this.
| Partition | Minimum Allocation | Maximum Allocation | Partition Type | Partition Name |
| --------- | ------------------ | ------------------ | -------------- | -------------- |
| Boot partition | Default | 1024MiB | EF00 | boot |
| Swap partition | Default | 16GiB | 8200 | swap |
| Root partition | Default | 32GiB | Default | root |
| Home partition | Default | The Remainder of the drive | Default | home |

Help: EF00 = uefi bootable partition, 8200 = swap and 8300 = linux filesystem.<br>
Note: If you get something above the boot partition with 1000KiB of free space, DON'T TOUCH IT. That is the protective MBR allocation.

f. write changes to disk


## 4. Format and Mount your partitions.
a. Run `lsblk` to see what your doing.<br>
b. Run `mkfs.ext4 /dev/root_partition` to format the root partition and run `mount /dev/root_partition /mnt` to mount the drive.<br>
c. Run `mkfs.ext4 /dev/home_partition` to format the home partition and run `mount --mkdir /dev/home_partition /mnt/home` to create and mount the home partition.<br>
d. Run `mkswap /dev/swap_partition` to format the swap partition and run `swapon /dev/swap_partition` to enable swap.<br>
e. Run `mkfs.fat -F 32 /dev/efi_system_partition` to format the boot partition and run `mount --mkdir /dev/efi_system_partition /mnt/boot` to create and mount the boot partition.


## 5. Configure Mirrorlist.
This step isn't really necessary but I would highly recommend it as it sorts the servers from best to worst.

Note: Since we don't have a GUI interface for file management we must do everything through command line.

a. Run `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup` to create a backup.<br>
b. Run `nano /etc/pacman.d/mirrorlist` to see if all the servers that are listed in the file are uncommented.<br>
c. Once exited nano, run `pacman -Sy` to update the package database on the ISO.<br>
d. Run `pacman -S pacman-contrib` to install the tools needed for sorting the servers.<br>
e. Run `rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist` to sort the servers in the backup file and copy it to the main file.


## 6. Download/Installing Essential Packages.
This step is where you get to actually install your system.

The following packages that it will install are the necessary core packages and the drivers for some wifi cards and sound cards.

Run `pacstrap -K /mnt base base-devel linux linux-headers linux-firmware linux-firmware-marvell linux-firmware-whence man-db man-pages nano sof-firmware` to install the packages.


## 7. Generating the fstab and chrooting into the install.
This step is where you will generate the partition UUID as without doing so will result in a system that wont know what it's doing.

a. Run `genfstab -U /mnt >> /mnt/etc/fstab` to generate the fstab file.<br>
b. Run `arch-chroot /mnt` to gain access to your install.


## 8. Localisation and Timezone.
This step is to tell Arch Linux where you are from so that the locale and timmezone will be set accordingly. It will also set the system clock.

WARNING: Anything and everything in this part is important. If you mess up when entering these commands your install is dead.

a. Run `nano /etc/locale.gen` and scroll down to your locale and uncomment it. If you don't know your locale then uncomment `en_US.UTF-8 UTF-8`.<br>
b. Once your locale has been uncommented, run `locale-gen` to generate the locale files.<br>
c. Even though you've already assigned the locale, you still need to echo the locale for older programs to function properly. To do this run `echo "LANG=[the locale you selected].UTF-8" >> /etc/locale.conf` to set the legacy locale.<br>
d. This step is important and should be done either way. Run `export LANG=[the locale you selected].UTF-8`.<br>
e. Skip this step if you have a us keyboard layout. If you have a keyboard other than us run `echo "KEYMAP=[your keyboard layout]" >> /etc/vconsole.conf`.<br>
f. To set the timezone, run `ls /usr/share/zoneinfo` to list the unix timezones.<br>
g. Once you have found your timezone, run `ln -sf /usr/share/zoneinfo/[Your Country Here]/[Your Timezone Here] /etc/localtime` to add a symbolic link for your timezone.<br>
h. To link the software clock to the hardware clock, run `hwclock --systohc` to set the hardware clock.


## 9. Configure Pacman/Package Manager.
This step is where you will configure pacman to be able to download faster and also enable the ability to download 32-bit packages through the Multilib repository.

a. Run `nano /etc/pacman.conf` to enter the pacman config file.<br>
b. Uncomment the line that you see below.<br>

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

c. In the Misc Options area, add/uncomment the following items. `ParallelDownloads = 5`, `Color` and `ILoveCandy`.<br>
d. Once saved run `pacman -Sy` to apply the modified config and to download the repo.


## 10. Installing more packages and enabling system services.
This step is where you are going to install some more packages and some miscellaneous drivers for connecting internet as well as enabling some necessary system functions.

Note: Skip the fstrim function if you don't have an SSD.

a. Run `pacman -S git networkmanager reflector pacman-contrib bash-completion` to install the listed packages.<br>
b. To make sure that your CPU has no active exploits on it's firmware, you need to install the microcode. To install your CPU's microcode run `pacman -S [Your CPU Brand]-ucode`.<br>
c. Enable the following services to start the drivers and system functions.
```
systemctl enable NetworkManager.service
systemctl enable fstrim.timer
systemctl enable reflector.timer
```

## 11. Hostname Configuration and User Setup.
This step is where you will set the computer name and add your user accounts. 

a. Run `echo "[Insert Computer Name Here]" >> /etc/hostname` to set the computer name.<br>
b. Run `nano /etc/hosts` and add the following into the file.
```
127.0.0.1        localhost
::1              localhost
127.0.1.1        [Add same hostname as before.]
```

c. To setup the administrator account, run `passwd` to set the root password.<br>
d. To add a user account, run `useradd -m -G wheel,storage,power -s /bin/bash [Insert Username Here]` to create your user account.<br>
e. Run `passwd [Insert Username Here]` to set the password for the user account that you just created.<br>
f. Run `EDITOR=nano visudo` to and edit the following permissions.<br>
Uncomment `%wheel ALL=(ALL) ALL` and add `Defaults rootpw` to the bottom of the file.

Note: The `Defaults rootpw` is so that you use the root password instead of your user password for sudo. (makes more like windows)


## Bootloader.
Here is a little choose your own adventure bit for this install guide. There are three commonly used boot loaders that people tend to install. Choose the one you prefer and forget about the other one.

GRand Unified Bootloader / GRUB. (Easy Mode)<br>
This bootloader is the most common across most distro's and has the most documentation around customisation.<br>

rEFInd. (Medium Mode)<br>
This bootloader was originally designed for dualbooting mac but has been improved to support many more. This is what I would recommend if your trying to dualboot windows for some reason. (I'm not helping with setting up windows dualboot).

Systemd-Boot. (Hard Mode)<br>
This bootloader is what I would recommended you as the packages are preinstalled onto your system during install.

If you chose to install GRUB then ONLY do 12a.<br>
If you chose to install rEFInd then ONLY do 12b.<br>
If you chose to install Systemd-Boot then ONLY do 12c.

### 12a. GRUB. (Linux dual boot/Easy Mode)
a. Run `pacman -S grub efibootmgr` to install the necessary packages.<br>
b. Run `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` to inject and install GRUB to your system.<br>
c. Run `grub-mkconfig -o /boot/grub/grub.cfg` to generate the configuration files.

### 12b. rEFInd. (Medium Mode)
a. Run `pacman -S refind` to install the necessary packages.<br>
b. Run `refind-install` to inject and install rEFInd to your system.<br>
c. Run `nano /boot/refind_linux.conf` and modify the "Boot with standard options" line so that it has `initrd=[Your CPU Brand]-ucode.img` at the end.

### 12c. Systemd-Boot. (Requires manual entries/Hard Mode)
a. Run `ls /sys/firmware/efi/efivars` to verify if the efi firmware is mounted and installed.<br>
b. Run `bootctl install` to inject and install Systemd-Boot to your system.<br
c. Run `nano /boot/loader/entries/arch.conf` and add the following lines.
```
title Arch Linux
linux /vmlinuz-linux (change this depending on what kernel you have).
initrd /initramfs-linux.img
initrd /[Your CPU Brand]-ucode.img
```

e. Once added everything into the file, run `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition) rw" >> /boot/loader/entries/arch.conf` to add the partition UUID for the root partition. This is important as it tells Arch Linux to only boot to that drive. (Credit to Glorious Eggroll for this command.)


## 13. Graphics Drivers
This step is what I like to call "NIGHTMARE MODE" as you will be installing your GPU drivers. The drivers have been sorted based on what manufacturer your card is from. So select the one that matches your card.

Note: There are two NVIDIA drivers so PLEASE be careful when installing your GPU driver. 

| Manufacturer | Instructions |
| ------------ | ------------ |
| AMD | Run `pacman -S xf86-video-amdgpu libva-mesa-driver mesa rocm-opencl-runtime vulkan-radeon lib32-libva-mesa-driver lib32-mesa lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for AMD cards. |
| INTEL | Run `pacman -S xf86-video-intel mesa intel-compute-runtime intel-media-driver vulkan-intel lib32-mesa lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for INTEL cards. |
| NVIDIA (PROPRIETARY) | Run `pacman -S nvidia-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for MAXWELL series cards or newer. |
| NVIDIA (Open GPU Kernel Modules) | Run `pacman -S nvidia-open-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for TURING series cards or newer. |


## Configure Drivers for KMS/Wayland Support.
This step should only be done with the version that matches your card.

If you have installed the NVIDIA drivers then ONLY do 14a.<br>
If you have installed the AMDGPU drivers then ONLY do 14b.<br>
If you have installed the INTEL drivers then ONLY do 14c.

WARNING: If for whatever reason you mess up on this step your system is dead!

### 14a. NVIDIA
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)`<br>
b. Once those modules have been added, run `mkinitcpio -P` to regenerate the kernel initramfs.<br>
c. This step will vary depending on your bootloader so make sure you select the correct one.

| Bootloader | Instructions |
| ---------- | ------------ |
| GRUB | 1. Run `nano /etc/default/grub` and modify the `GRUB_CMDLINE_LINUX_DEFAULT=` line to look like this. `GRUB_CMDLINE_LINUX_DEFAULT=... nvidia-drm.modeset=1`.<br>2.Once added, run `grub-mkconfig -o /boot/grub/grub.conf` to regenerate the grub configuration files. |
| rEFInd | Run `nano /boot/refind_linux.conf` and at the end of the "Boot with standard options" line add `nvidia-drm.modeset=1`. |
| Systemd-Boot | Run `nano /boot/loader/entries/arch.conf` and at the end of the options line add `nvidia-drm.modeset=1`. |

### 14b. AMDGPU
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... amdgpu ...)`<br>
b. Once this module has been added, run `mkinitcpio -P` to regenerate the kernel initramfs.<br>

### 14c. INTEL
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... i915 ...)`<br>
b. Once this module has been added, run `mkinitcpio -P` to regenerate the kernel initramfs.<br>


## 15. Unmount drives and Reboot system.
a. Type `exit` to return back to the install drive.<br>
b. Type `umount -r /mnt` to safely unount the partitions.<br>
c. `reboot`<br>
Congratulations. You have sucessfully installed the base version of Arch Linux. However you're not done just yet.


## 16. General First Install Checks.
This step is just a general after installation check to make sure that nothing went wrong with the install. It also contains configuration for regenerating mirrorlists automatically.

Note: Now that you're actually using your system now, you will need to use sudo to perform root privilages.

a. Once booted up and logged in, run `systemctl --failed` to verify a sucessful bootup.<br>
b. Run `sudo reflector --country [Your Country Here] --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist` to set refector to generate the mirrorlist based on the settings given.<br>
c. Run `sudo nano /etc/xdg/reflector/reflector.conf` and make sure the file is configured to your liking. An example has been provided below:
```
--country [Your Country Here]
--age 12
--protocol https
--sort rate
--save /etc/pacman.d/mirrorlist
```

d. Run `sudo pacman -Sy` to resync and update the servers.


## 17. Enabling AUR support and flatpak.
This step is necessary if you want to use the best part of Arch linux. The Arch User Repository. This will also install flatpak.

Traditionally, if you want to install packages from the AUR, you would need to compile them from source but with an AUR Helper it builds and installs everything for you.

a. Run `git clone https://aur.archlinux.org/paru.git` to download the required files to compile Paru.<br>
b. Run `cd paru` to go into the folder.<br>
c. Run `makepkg -si` to install Paru.<br>
d. Once Paru is installed run `paru -Syyu` to update all packages installed on your computer.<br>
e. Run `sudo pacman -S flatpak` to install the flatpak repo and installer.<br>
f. `reboot` system to complete the install of flatpak.


## 18. Graphical Environment.
This step is probably the most confusing to new users. (It was also the most difficult part of the rewrite).

Currently, there are two well known video drivers for linux. Wayland and Xorg (legacy). This guide is mainly focused on Xorg, however, if you want to use Wayland then it's already enabled and ready to go.

If you do not want to use Xorg at all and want to have a pure Wayland configuration, then skip part 1 and just select a Wayland based desktop environment.

Note: Most wayland compositors may not work with Nvidia, so if you have Nvidia use Xorg.

### Part 1. Installing Xorg.
Run `sudo pacman -S xorg xorg-xinit` to install the xorg video drivers.

### Part 2. Selecting your Desktop Environment and or Window Manager.
| Xorg Desktop Environment | Instructions |
| ------------------------ | ------------ |
| AwesomeWM | Run `sudo pacman -S awesome alacritty pcmanfm-qt` to install the packages for a working install of AwesomWM. |
| DWM | Note: You MUST configure the packages `config.h` file before building the package.<br>1. Run `git clone https://aur.archlinux.org/dwm.git`to download the PKGBUILD.<br>2. Configure the `config.h` file to your liking and ensure there are no errors.<br>3. Run `makepkg -si` to build and install your configured copy of DWM. |
| i3 | Run `sudo pacman -S i3 alacritty pcmanfm-qt dmenu` to install the packages for a working install of i3. |
| LXQt | Run `sudo pacman -S lxqt breeze-icons network-manager-applet leafpad` to install the packages for a working install of LXQt. |
| Xfce | Run `sudo pacman -S xfce xfce-goodies network-manager-applet` to install the packages for a working install of Xfce. |

| Wayland Desktop Environments | Instructions |
| ---------------------------- | ------------ |
| Gnome | Run `sudo pacman -S gnome gnome-tweaks xdg-desktop-portal-gnome` to install the packages for a working install of Gnome. |
| KDE Plasma | Run `sudo pacman -S plasma kde-applications qt5-wayland xdg-desktop-portal-kde` to install the packages for a working install of KDE Plasma. When prompted, select the VLC backend for audio. |
| Sway | Run `sudo pacman -S sway swaylock swayidle swaybg waybar mako polkit-kde-agent qt5-wayland qt6-wayland cliplist light grim slurp foot xdg-desktop-portal-wlr` to install most of the packages reqired for a working install of Sway.<br>With Paru, run `paru -S tofi` to install the application launcher. |

### Part 3. Installing and enabling a display manager.
| Display Manager | Instructions |
| --------------- | ------------ |
| GDM | Note: Since GDM is included with Gnome you don't need to install anything.<br>To enable the Display Manager upon reboot run `sudo systemctl enable gdm.service`. |
| SDDM | Run `sudo pacman -S sddm` to install SDDM and then run `sudo systemctl enable sddm.service` to enable the Display Manager upon reboot. |
| LightDM | Run `sudo pacman -S lightdm` to install the base version of lightDM and run `sudo systemctl enable lightdm.service`  to enable the Display Manager upon reboot.<br>Since LightDM does not include a environment to run on you wil have to install one of the greeters listed below. |
| StartX | Since StartX is kind of difficult to setup i will simply like to the [Arch Wiki](https://wiki.archlinux.org/title/Xinit#Autostart_X_at_login) for instructions. |
| wlroots on TTY | Since most wayland compositors are based on wlroots, they do not allow launching with a Display Manager. So, I will simply link to the [Arch Wiki](https://wiki.archlinux.org/title/Sway#Automatically_on_TTY_login) for instructions on how to setup TTY login. |

### (Only for LightDM) Part 4. Choose the greeter you want to use for LightDM.
If your using any other display manager then you can skip this step.

The two versions is just what style you want. If you want a style that looks like Gnome then select the GTK version. If you want a style thats easy to configure and looks great then use the Webkit2 version.

| Greeter | Instructions |
| ------- | ------------ |
| GTK | Run `sudo pacman -S lightdm-gtk-greeter lightdm-gtk-greeter-settings` to install the GTK greeter and configurator tool. |
| Webkit2 | Run `sudo pacman -S lightdm-webkit2-greeter` to install the webkit2 greeter. |


## 19. Zsh Setup and Configuration.
This step is if you want a different terminal shell from the default bash setup.

Note: NEVER USE A ZSH PLUGIN MANAGER AS IT IS JUST BLOATWARE!!!!

Tip: You might want to move some code from the `.bashrc` file to the `.zshrc` file (e.g. the prompt and the aliases). It's also recommended to move code from the `.bash_profile` file to the `.zprofile` file (e.g. the code that makes your window manager work).

a. Run `sudo pacman -S zsh zsh-completions` to install Zsh.<br>
b. Once installed, run `zsh` to begin the initial setup<br>
c. Now that Zsh is configured, run `chsh -s /usr/bin/zsh` to set Zsh as your default terminal shell.


## 20. Audio Drivers.
This step is necessary if you want to have a working audio setup. 

Note: One of the packages, `pipewire` to be exact, is required for wayland since by itself wayland does NOT allow screen capture for programs.

Run `sudo pacman -S alsa-ucm-conf alsa-utils alsa-plugins pavucontrol pipewire pipewire-audio pipewire-alsa pipewire-jack pipewire-pulse lib32-pipewire lib32-pipewire-jack pulsemixer qpwgraph wireplumber` to install all the packages needed for a working audio setup.


## 21. Gstreamer Full Support. (Optional)
This step only applies to users who want Desktop Environments that don't utilise VLC. Window Managers and KDE with VLC backend can go without this though.

Run `sudo pacman -S gstreamer lib32-gstreamer gst-libav gst-plugins-bad gst-plugins-base gst-plugins-good gst-plugins-ugly gst-plugins-pipewire gstreamer-vaapi` and `paru -S gst-plugin-libde265 gst-plugins-openh264` to install the base package and other codec's.


## 22. Reboot and login.
Run `reboot`, then login to your user account and then you should see the Desktop you installed.<br>
Congratulations You have sucessfully installed Arch Linux.


## Applications.
This is a list of all programs that have linux support that I am aware of. There are games and other programs in here too.

Note 1: This list is only if your using the terminal for installing packages and before you install a program, always remember to run a `sudo pacman -Sy` or `sudo pacman -Syu` to make sure the repos are up to date so that there is no incompatibility.

Note 2: If a program is distributed as an appimage, please use AppImageLauncher to install it instead of running it manually.

This list has been seperated into multiple sections based on what the package relates to.

| Essential Packages | Commands |
| ------------------ | -------- |
| AppImageLauncher | `paru -S appimagelauncher` |
| 7-Zip | `paru -S 7-zip-full` |
| Windows 11 Fonts | `paru -S ttf-ms-win11-auto` |
| Timeshift | `sudo pacman -S timeshift` |
| Downgrade | `paru -S downgrade` |
| Bluetooth | 1. `sudo pacman -S bluez bluez-utils`<br>2. `sudo systemctl enable bluetooth.service` |

| Game Launchers | Commands |
| ----- | -------- |
| Steam | `sudo pacman -S steam` |
| Steam Native Runtime Replacement | `sudo pacman -S steam-native-runtime` |
| Lutris | `sudo pacman -S lutris`<br>Note: Lutris requires you to have already installed the base version of Wine |
| YARG | 1. [Download on Github](https://github.com/YARC-Official/YARC-Launcher)<br>2. `sudo pacman -S hidapi systemd-libs` |
| Heroic Games Launcher | `paru -S heroic-games-launcher-bin` |
| Minecraft | `paru -S minecraft-launcher`<br>Note: Minecraft requires java 21 lts for builds from 1.21 onwards and java 8 lts can be used for any builds from classic to 1.12. |
| Prism Launcher (Minecraft) | `paru -S prismlauncher` |
| Lunar Client (Minecraft) | [Download from website](https://www.lunarclient.com/download) |

| Games | Commands |
| -------------- | -------- |
| osu! | `paru -S osu-laser-bin` |
| Katawa Shoujo | `paru -S katawa-shoujo` |
| Clone Hero v1.0.0.4080-final | `paru -S clonehero` |
| Clone Hero v1.1.0.4261-PTB | `paru -S clonehero-ptb` |
| Roblox (Grapejuice) | `paru -S grapejuice`<br>Note: Grapejuice requires you to have already installed the base version of Wine |
| Tentacle Locker 2 | [Download on itch](https://hotpink.itch.io/tl2) |
| Tentacle Locker | [Download on itch](https://hotpink.itch.io/tentacle-locker)<br>Note: Needs to be run through Wine |
| Protecc Your Loli | [Download on itch](https://kamuo.itch.io/proteccloli)<br>Note: Needs to be run through Wine |
| Doki Doki Literature Club | [Download on itch](https://teamsalvato.itch.io/ddlc) |
| Monika After Story Mod | [Download on Github](https://www.monikaafterstory.com/) |
| MonikA.I | [Download on Github](https://github.com/Rubiksman78/MonikA.I) |

| Emulators | Commands |
| --------- | -------- |
| Dolphin Emulator (Arch Package) | `sudo pacman -S dolphin-emu` |
| pcsx2 (Upstream) | `flatpak install pcsx2` |
| rpcs3 (Upstream) | `paru -S rpcs3-git` |
| DuckStation (Upstream) | `flatpak install duckstation` |
| melonDS (Upstream) | `flatpak install melonds` |
| Ryujinx (Upstream) | `flatpak install ryujinx` |
| CEMU (Upstream) | `flatpak install cemu` |
| mGBA (Arch Package) | `sudo pacman -S mgba-qt` |
| Snes9x (Arch Package) | `sudo pacman -S snes9x-gtk` |
| Panda3DS (Upstream) | [Download on Github](https://github.com/wheremyfoodat/Panda3DS) |
| ñ (PabloMK7 Fork) | Figure it out yourself |
| suyu (Upstream) | [Download on Git](https://git.suyu.dev/suyu/suyu) |

| Internet | Commands |
| -------- | -------- |
| Firefox | `sudo pacman -S firefox` |
| Chromium | `sudo pacman -S chromium` |
| Brave | `paru -S brave-bin` |
| Librewolf | `paru -S librewolf-bin` |

| Media | Commands |
| ----- | -------- |
| Ani-Cli | `paru -S ani-cli` |
| MPV | `sudo pacman -S mpv` |
| VLC | `sudo pacman -S vlc` |
| VLC-luajit | `paru -S vlc-luajit` |
| GoXLR-Utility | `paru -S goxlr-utility` |
| Physical Media | `sudo pacman -S libcdio libdvdread libdvdcss libdvdnav libbluray libaacs`<br>Note: If your using KDE applications to play cd's, run `sudo pacman -S audiocd-kio` to install the package. |

| Compatibility Tools/Wine | Commands |
| ------------------------ | -------- |
| Proton-GE | [Download on Github](https://github.com/GloriousEggroll/proton-ge-custom) |
| Wine-GE | [Download on Github](https://github.com/GloriousEggroll/wine-ge-custom) |
| GameMode | `sudo pacman -S gamemode lib32-gamemode` |
| Protonup-QT | `paru -S protonup-qt` |
| Wine | Please note that wine is literally a dependency nightmare if you don't know what you are doing.<br>1. `sudo pacman -S wine-staging winetricks`<br>2. `sudo pacman -S --needed alsa-lib alsa-plugins cups dosbox ffmpeg giflib gnutls gst-plugins-base-libs gtk3 lib32-alsa-lib lib32-alsa-plugins lib32-giflib lib32-gnutls lib32-gst-plugins-base-libs lib32-gtk3 lib32-libpulse lib32-libva lib32-libxcomposite lib32-libxinerama lib32-ocl-icd lib32-sdl2 lib32-v4l-utils lib32-vulkan-icd-loader libgphoto2 libpulse libva libxcomposite libxinerama ocl-icd samba sane sdl2 v4l-utils vulkan-icd-loader` |
| WineASIO | This package is good for if you plan on running Ableton or FL Studio in Wine<br>1. `paru -S wineasio`<br>2. `sudo usermod -aG realtime $(whoami)`<br>For 64bit, run: `wine64 regsvr32 /usr/lib/wine/x86_64-windows/wineasio64.dll`<br>For 32bit, run: `regsvr32 /usr/lib/wine/i386-windows/wineasio32.dll` |

| Miscellaneous | Commands |
| ------------- | -------- |
| Thunderbird | `sudo pacman -S thunderbird` |
| qBittorrent | `sudo pacman -S qbittorrent` 
| Discord | `flatpak install discord` |
| OpenRGB | 1. `paru -S openrgb`<br>2. `sudo pacman -S i2c-tools` |
| Inochi2D Session | [Download on Github](https://inochi2d.com/) |
| Rofi | `sudo pacman -S rofi` |
| Zsh plugins | 1. `sudo pacman -S zsh-syntax-highlighting zsh-autosuggestions`<br> 2. `echo "source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> .zshrc` and `echo "source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh" >> .zshrc` |
| Syncthing | `sudo pacman -S syncthing` |
| Bridge/Enchor.us | [Download on Github](https://github.com/Geomitron/Bridge) |

| Programming | Commands |
| ------------ | -------- |
| Python | `sudo pacman -S python python-pip` |
| NodeJS | `sudo pacman -S nodejs-lts-iron npm` |
| Zulu Java8 | `paru -S zulu-8-bin` |
| Zulu Java17 | `paru -S zulu-17-bin` |
| VS Code | 1. `paru -S visual-studios-code-bin`<br>2. `sudo pacman -S dotnet-runtime dotnet-sdk aspnet-runtime mono-msbuild mono-msbuild-sdkresolver mono` |

| Production | Commands |
| ---------- | -------- |
| Reaper DAW | `sudo pacman -S reaper` |
| Polyphone | `sudo pacman -S polyphone` |
| Audacity | `sudo pacman -S audacity` |
| Moonscraper Chart Editor | [Download on Github](https://github.com/FireFox2000000/Moonscraper-Chart-Editor) |
| Blender | `sudo pacman -S blender` |
| Blender 2.79b | [Download on website](https://download.blender.org/release/Blender2.79/) |
| Unreal Engine | Figure it out yourself |
| Inochi2D Creator | [Download on Github](https://inochi2d.com/) |
| OBS Studio Tytan652 | 1. `paru -s obs-studio-tytan652`<br>2. `sudo pacman -S v4l2loopback-dkms` |
| Kame-Editor | `paru -S kame-editor-git` |

| Joke Packages | Commands |
| ------------- | -------- |
| cMatrix | `sudo pacman -S cmatrix` |
| cowsay | `sudo pacman -S cowsay` |
| lolcat | `sudo pacman -S lolcat` |
| neofetch (legacy) | `sudo pacman -S neofetch` |
| HyFetch (Updated Neofetch fork) | Note: This fork requires you to replace the neofetch command with neowofetch to use the updated fork.<br>`sudo pacman -S hyfetch` |
| Fastfetch | Note: I would recommend you use this fork as it works better and is more feature complete.<br>`sudo pacman -S fastfetch` |
| Activate Linux | `paru -S activate-linux-git` |
| Arch Linux Wallpapers | This isn't a joke package. It's literally just some Arch Linux themed wallpapers.<br>`sudo pacman -S archlinux-wallpaper` |

| Flatpak Packages | Commands |
| ---------------- | -------- |
| Flatseal | `flatpak install flatseal` |
| OBS Studio | 1. `flatpak install obs-studios`<br>2. `sudo pacman -S v4l2loopback-dkms` |
| Extension Manager | This package is ONLY for Gnome.<br>`flatpak install ExtensionManager` |
| Bottles | `flatpak install bottles`<br>Note: Bottles requires you to have already installed the base version of Wine |

| System Diagnostic Tools | Commands |
| ----------------------- | -------- |
| Mangohud | `sudo pacman -S mangohud lib32-mangohud` |
| GOverlay | `paru -S goverlay-bin` |
| Btop++ | `sudo pacman -S btop` |
| Htop | `sudo pacman -S htop` |

| Device Hacking | Commands |
| -------------- | -------- |
| Wireshark | `sudo pacman -S wireshark-qt` |
| Santroller Configurator | [Download on Github](https://github.com/Santroller/Santroller) |
| Fusée Launcher Interfacée | `paru -S fusee-interfacee-tk-bin` |
| OSCDL | [Download on Github](https://github.com/dhtdht020/osc-dl) |
| WiiUDownloader | [Download on Github](https://github.com/Xpl0itU/WiiUDownloader) |
