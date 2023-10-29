# Arch Linux UEFI Install Guide

Small note: This took soo long to rewrite this guide as there is soo much stuff in it that had to be rewritten from scratch.

This guide assumes that your default language is english and you are on desktop with an AMD. NVIDIA and INTEL ARC is supported however it's also highly discouraged as the drivers can be weird.


## 0. Getting the ISO
a. Go to archlinux.org and click on download.<br>
b. Download the image from the download server that is in you region.<br>
c. Get some kind of ISO burner for a usb and just write the ISO to the USB.<br>
Note: If you have qbittorrent, you can use the torrent link instead of the download mirrors.


## 1. Basic initial Setup
a. Run the command `cat /sys/firmware/efi/fw_platform_size` to check if your booted into UEFI.<br>
b. Run `timedatectl status` to ensure the date and time is accurate.


## 2. Networking
If you're not going to be using WI-FI, then skip part 1.

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
b. Run `mkswap /dev/swap_partition` to format the swap partition and run `swapon /dev/swap_partition` to enable swap.<br>
c. Run `mkfs.ext4 /dev/root_partition` to format the root partition and run `mount /dev/root_partition /mnt` to mount the drive.<br>
d. Run `mkfs.ext4 /dev/home_partition` to format the home partition and run `mount --mkdir /dev/home_partition /mnt/home` to create and mount the home partition.<br>
e. Run `mkfs.fat -F 32 /dev/efi_system_partition` to format the boot partition.<br>
f. Run `mount --mkdir /dev/efi_system_partition /mnt/boot` to create and mount the boot partition.


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

Run `pacstrap -K /mnt base base-devel linux linux-headers linux-firmware linux-firmware-marvell linux-firmware-whence man-db man-pages nano sof-firmware texinfo` to install the packages.


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
e. Skip this step if you have a us keyboard layout. If you have a keyboard other than us run `echo "KEYMAP=[your keyboard layout]" >> /etc/vconsole.conf`.<br>
f. Run `ls /usr/share/zoneinfo` to list the unix timezones.<br>
g. Once found your timesone run `ln -sf /usr/share/zoneinfo/[Your Country Here]/[Your Timesone Here] /etc/localtime` to add a symbolic link for your time.<br>
h. Run `hwclock --systohc` to set the hardware clock.


## 9. Configure Pacman/Package Manager.
This step is where you will configure pacman to be able to download faster and enable the Multilib repo for 32-bit programs.

a. Run `nano /etc/pacman.conf` to enter the pacman config file.<br>
b. Uncomment the line that you see below.<br>

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

c. In the Misc Options area, add/uncomment the following items. `ParallelDownloads = 5`, `Color` and `ILoveCandy`.<br>
d. Once saved run `pacman -Sy` to update and download the repos.


## 10. Installing more packages and enabling system services.
This part is where you're going to install some more packages and some drivers for basic networking and enable some system functions.

Note: If you have an AMD processor, then you can skip the microcode installation, however I would still recommend installing it anyways.

a. Run `pacman -S git networkmanager reflector pacman-contrib bluez bluez-utils bash-completion` to install the packages.<br>
b. To make sure your CPU has no active exploits on it firmware, you need to install the microcode. To install the CPU microcode run `pacman -S [Your CPU Brand]-ucode`.<br>
c. Enable the following services to start the drivers.
```
systemctl enable bluetooth.service
systemctl enable NetworkManager.service
systemctl enable fstrim.timer
systemctl enable reflector.timer
```


## 11. Hostname Configuration and User Setup.
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

The `Defaults rootpw` is so that you use the root password instead of your user password for sudo. (makes more like windows)


## Bootloader.
Here is a little choose your own adventure bit for this install guide. There are two commonly boot loaders that people tend to install. Choose the one you prefer and forget about the other one.

Systemd-Boot. (Hard Mode)<br>
This bootloader is what I would recommended you as the packages are preinstalled onto system during install.

GRand Unified Bootloader / GRUB. (Easy Mode)<br>
This bootloader is the most common across most distro's and has the most documentation.<br>

If you chose to install GRUB then ONLY do 15a.<br>
If you chose to install Systemd-Boot then ONLY do 15b.

### 12a. GRUB. (Linux dual boot/Easy Mode)
a. Run `pacman -S grub efibootmgr` to install the necessary packages.<br>
b. Run `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` to inject and install GRUB to your system.<br>
c. Run `grub-mkconfig -o /boot/grub/grub.cfg` to generate the configuration files.

### 12b. Systemd-Boot. (Requires manual entries/Hard Mode)
a. Run `ls /sys/firmware/efi/efivars` to verify if the efi firmware is mounted and installed.<br>
b. Run `bootctl install` to inject and install Systemd-Boot to your system.<br>
c. Run `systemctl enable systemd-boot-update.service` to enable the updater script.<br>
d. Run `nano /boot/loader/entries/arch.conf` and add the following lines.
```
title Arch Linux
linux /vmlinuz-linux (change this depending on what kernel you have).
initrd /initramfs-linux.img
initrd /intel-ucode.img or /amd-ucode.img
```

e. Once added everything into the file, run `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition) rw" >> /boot/loader/entries/arch.conf` to add the partition id of the root partition so that it tells Arch Linux that it will boot to that drive only. (Credit to Glorious Eggroll for this command)


## 13. Graphics Drivers
This step is necessary if you want your GPU to be working properly. The drivers are sorted based on what manufacturer your card is from so select the one that matches your card.

| Manufacturer | Instructions |
| ------------ | ------------ |
| AMD | Run `pacman -S xf86-video-amdgpu libva-mesa-driver mesa rocm-opencl-runtime vulkan-radeon lib32-libva-mesa-driver lib32-mesa lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for AMD cards. |
| INTEL | Run `pacman -S xf86-video-intel mesa intel-compute-runtime intel-media-driver vulkan-intel lib32-mesa lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for INTEL cards. |
| NVIDIA (PROPRIETARY) | Run `pacman -S nvidia-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for MAXWELL series cards or newer. |
| NVIDIA (Open GPU Kernel Modules) | Run `pacman -S nvidia-open-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers for TURING series cards or newer. |


## Configure Drivers for KMS/Wayland Support.
This part should only be done with the version that matches your card.

WARNING: If you for whatever reason mess up on this your system is bricked!

### 14a. NVIDIA
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

### 14b. AMDGPU
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... amdgpu ...)`<br>
b. Once added those modules, run `mkinitcpio -P` to regenerate the initramfs.

### 14c. INTEL
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... i915 ...)`<br>
b. Once added those modules, run `mkinitcpio -P` to regenerate the initramfs.


## 15. Unmount drives and Reboot system.
Congratulations. You have sucessfully installed the basic form of Arch Linux. However you're not done just yet.

a. Type `exit` to return back to the install drive.<br>
b. Type `umount -r /mnt` to safely unount the partitions.<br>
c. `reboot`


## 16. General First Install Checks.
This part is just a general after install checks to make sure nothing went wrong with the install. It also contains configuration for regenerating mirrorlists automatically.

Note: Now that you're actually using your system now, you will need to use sudo to perform root privilages.

a. Once booted up, run `systemctl --failed` to verify a sucessful bootup.<br>
b. Run `sudo reflector --country [Your Country Here] --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist` to set refector to generate the mirrorlist based on the settings given.<br>
c. Run `sudo nano /etc/xdg/reflector/reflector.conf` and make sure the file is configured to your liking. An example is provided below:
```
--country [Your Country Here]
--age 12
--protocol https
--sort rate
--save /etc/pacman.d/mirrorlist
```

d. Run `sudo pacman -Sy` to resync and update the servers.


## 17. Enabling AUR support and flatpak.
This step is necessary if you want to use the Arch User Repository for community maintained packages. 

Traditionally, if you want to install packages from the AUR, you would need to compile them from source but with an AUR Helper it builds and installs it for you.

a. Run `git clone https://aur.archlinux.org/paru.git` to download the required files to compile Paru.<br>
b. Run `cd paru` to go into the folder.<br>
c. Run `makepkg -si` to install Paru.<br>
d. Once Paru is installed run `paru -Syyu` to update all packages installed on your computer.<br>
e. Run `sudo pacman -S flatpak` to install the flatpak repo and installer.<br>
f. `reboot` system to complete the install of flatpak.


## 18. Graphical Environment.
This step is probably the most confusing to users and the most difficult part of the rewrite for me.

Currently, there are two well known video drivers for linux. Wayland and Xorg. This guide is mainly focused on Xorg, however, if you want to use Wayland then it's already enabled and ready to go.

If you do not want to use Xorg at all and want to have a pure Wayland configuration, then skip part 1 and just select a Wayland based desktop environment.

Note: If you are having an issue with wayland on NVIDIA GPU's, install the `egl-wayland` package to fix the issue.

### Part 1. Installing Xorg.
Run `sudo pacman -S xorg xorg-xinit` to install the xorg video drivers.

### Part 2. Selecting your Desktop Environment and or Window Manager.
| Xorg Desktop Environment | Instructions |
| ------------------------ | ------------ |
| AwesomeWM | Run `sudo pacman -S awesome alacritty pcmanfm-qt` to install the packages for a working install of AwesomWM. |
| i3 | Run `sudo pacman -S i3 alacritty pcmanfm-qt dmenu` to install the packages for a working install of i3. |
| KDE Plasma (X11) | Run `sudo pacman -S plasma kde-applications` to install the packages for a working install of KDE Plasma. When prompted, select the VLC backend for audio. |
| LXQt | Run `sudo pacman -S lxqt breeze-icons network-manager-applet leafpad` to install the packages for a working install of LXQt. |
| Xfce | Run `sudo pacman -S xfce xfce-goodies network-manager-applet` to install the packages for a working install of Xfce. |

| Wayland Desktop Environments | Instructions |
| ---------------------------- | ------------ |
| Gnome | Run `sudo pacman -S gnome gnome-tweaks xdg-desktop-portal-gnome` to install the packages for a working install of Gnome. |
| Hyprland | Note: wlroots based compositors only works on AMDGPU's and only under Wayland.<br>Run `sudo pacman -S hyprland mako polkit-kde-agent cliphist grim slurp foot qt5-wayland qt6-wayland xdg-desktop-portal-hyprland` to install most of the packages reqired for a working install of Sway.<br>With Paru, run `paru -S tofi waybar-hyprland swww` to install the application launcher. |
| KDE Plasma (Wayland) | If you want to use KDE under Wayland, install the X11 version and then run `plasma-wayland-session qt6-wayland xdg-desktop-portal-kde` to install the wayland packages. When prompted, select the VLC backend for audio. |
| Sway | Note: wlroots based compositors only works on AMDGPU's and only under Wayland.<br>Run `sudo pacman -S sway swaylock swayidle swaybg waybar mako polkit-kde-agent qt5-wayland qt6-wayland cliplist light grim slurp foot xdg-desktop-portal-wlr` to install most of the packages reqired for a working install of Sway.<br>With Paru, run `paru -S tofi` to install the application launcher. |

### Part 3. Installing and enabling a display manager.
| Display Manager | Instructions |
| --------------- | ------------ |
| GDM | Note: Since GDM is included with Gnome you don't need to install anything.<br>To enable the Display Manager upon reboot run `sudo systemctl enable gdm.service`. |
| SDDM | Run `sudo pacman -S sddm` to install SDDM and then run `sudo systemctl enable sddm.service` to enable the Display Manager upon reboot. |
| LightDM | Run `sudo pacman -S lightdm` to install the base version of lightDM and run `sudo systemctl enable lightdm.service`  to enable the Display Manager upon reboot.<br>Since LightDM does not include a environment to run on you wil have to install one of the greeters listed below. |
| StartX | Since StartX is kind of difficult to setup i will simply like to the [Arch Wiki](https://wiki.archlinux.org/title/Xinit#Autostart_X_at_login) for instructions. |
| wlroots on TTY | Since most wayland compositors are based on wlroots, they do not allow launching with a Display Manager. So, I will simply link to the [Arch Wiki](https://wiki.archlinux.org/title/Sway#Automatically_on_TTY_login) for instructions on how to setup TTY login. |

### (Only for LightDM) Part 4. Choose the greeter you want to use for LightDM.
If your using any other display manager then you can skip this step and move onto Part 5.

The two versions is just what style you want. If you want a style that looks like Gnome then select the GTK version. If you want a style thats easy to configure and looks great then use the Webkit2 version.

| Greeter | Instructions |
| ------- | ------------ |
| GTK | Run `sudo pacman -S lightdm-gtk-greeter lightdm-gtk-greeter-settings` to install the GTK greeter and configurator tool. |
| Webkit2 | Run `sudo pacman -S lightdm-webkit2-greeter` to install the webkit2 greeter. |


## 19. Zsh Setup and Configuration.
This step is if you want a different terminal shell from the default bash setup.

Note: NEVER USE A ZSH PLUGIN MANAGER AS IT IS JUST BLOATWARE!!!!

a. Run `sudo pacman -S zsh zsh-completions` to install Zsh.<br>
b. Once installed, run `zsh` to begin the initial setup<br>
c. Now that Zsh is setup, run `chsh -s /usr/bin/zsh` to set Zsh as your default shell.


## 20. Audio Drivers.
This step is necessary if you want to have a working audio setup. 

Note: One of the packages, `pipewire` to be exact, is required for wayland since by itself wayland does NOT allow screen capture for programs, and the `alsa-ucm-conf` package is needed for basic non configurable GoXLR support.

Run `sudo pacman -S alsa-ucm-conf alsa-utils alsa-plugins pavucontrol pipewire pipewire-audio pipewire-alsa pipewire-jack pipewire-pulse lib32-pipewire lib32-pipewire-jack pulsemixer qpwgraph wireplumber` to install all the packages needed for a working audio setup.


## 21. Gstreamer Full Support. (Optional)
This step only applies to users who want Desktop Environments that don't utilise VLC . Window Managers and KDE with VLC backend can go without this though.

Run `sudo pacman -S gstreamer lib32-gstreamer gst-libav gst-plugins-bad gst-plugins-base gst-plugins-good gst-plugins-ugly gst-plugins-pipewire gstreamer-vaapi` and `paru -S gst-plugin-libde265 gst-plugins-openh264` to install the base package and other codec's.


## 22. Reboot and login.
Run `reboot`, then login to your user account and then you should see the Desktop you installed. Congratulations You have sucessfully installed Arch Linux.


## Applications.
This is a list of all programs that use linux that I am aware of. There are games and other programs in here too.

Note 1: This list is only if your using the terminal for installing packages and before you install a program, always run a sudo pacman -Sy or sudo pacman -Syu to make sure the repos are up to date so there is no incompatibility.

Note 2: If a program is distributed as an appimage, please use AppImageLauncher to install it instead of running it manually.

This list has been seperated into multiple sections based on what the package relates to.

| Essential Packages | Commands |
| ------------------ | -------- |
| AppImageLauncher | `paru -S appimagelauncher` |
| 7-Zip | `paru -S 7-zip-full` |
| Windows 11 Fonts | `paru -S ttf-ms-win11-auto` |
| Timeshift | `sudo pacman -S timeshift` |
| Downgrade | `paru -S downgrade` |

| Game Launchers | Commands |
| ----- | -------- |
| Steam | `sudo pacman -S steam` |
| Steam Native Runtime Replacement | `sudo pacman -S steam-native-runtime` |
| Lutris | `sudo pacman -S lutris`<br>Note: Lutris requires you to have already installed the base version of Wine |
| ScoreSpy | [Download from website](https://clonehero.scorespy.online/) |
| Heroic Games Launcher | `paru -S heroic-games-launcher-bin` |
| Minecraft | `paru -S minecraft-launcher`<br>Note: Minecraft requires java 17 lts for builds from 1.17 onwards and java 8 lts can be used for any older builds |

| Games | Commands |
| -------------- | -------- |
| osu! | `paru -S osu-laser-bin` |
| Katawa Shoujo | `paru -S katawa-shoujo` |
| Clone Hero v1.0.0.4080-final | `paru -S clonehero` |
| Roblox (Grapejuice) | `paru -S grapejuice`<br>Note: Grapejuice requires you to have already installed the base version of Wine |
| Tentacle Locker 2 | [Download on itch](https://hotpink.itch.io/tl2) |
| Tentacle Locker | [Download on itch](https://hotpink.itch.io/tentacle-locker)<br>Note: Needs to be run through Wine |
| Protecc Your Loli | [Download on itch](https://kamuo.itch.io/proteccloli)<br>Note: Needs to be run through Wine |
| Doki Doki Literature Club | [Download on itch](https://teamsalvato.itch.io/ddlc) |
| Monika After Story Mod | [Download on Github](https://www.monikaafterstory.com/) |
| MonikA.I | [Download on Github](https://github.com/Rubiksman78/MonikA.I) |

| Emulators | Commands |
| --------- | -------- |
| Dolphin Emulator | `paru -S dolphin-emu-beta-git` |
| pcsx2 | `flatpak install pcsx2` |
| rpcs3 | `paru -S rpcs3-git` |
| DuckStation | `flatpak install duckstation` |
| melonDS | `flatpak install melonds` |
| Yuzu | `flatpak install yuzu` |
| Ryujinx | `paru -S ryujinx-bin` |
| CEMU | `flatpak install cemu` |
| Citra | `flatpak install citra` |
| mGBA | `sudo pacman -S mgba-qt` |

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
| VLC-luajit | `paru -S vlc-luajit` |
| GoXLR-Utility | `paru -S goxlr-utility` |

| Compatibility Tools/Wine | Commands |
| ------------------------ | -------- |
| Proton-GE | [Download on Github](https://github.com/GloriousEggroll/proton-ge-custom) |
| Wine-GE | [Download on Github](https://github.com/GloriousEggroll/wine-ge-custom) |
| GameMode | `sudo pacman -S gamemode lib32-gamemode` |
| Protonup-QT | `paru -S protonup-qt` |
| Wine | Please note that wine is literally a dependency nightmare if you don't know what you are doing.<br>1. `sudo pacman -S wine-staging winetricks`<br>2. `sudo pacman -S --needed alsa-lib alsa-plugins cups dosbox ffmpeg giflib gnutls gst-plugins-base-libs gtk3 lib32-alsa-lib lib32-alsa-plugins lib32-giflib lib32-gnutls lib32-gst-plugins-base-libs lib32-gtk3 lib32-libpulse lib32-libva lib32-libxcomposite lib32-libxinerama lib32-ocl-icd lib32-sdl2 lib32-v4l-utils lib32-vulkan-icd-loader libgphoto2 libpulse libva libxcomposite libxinerama ocl-icd samba sane sdl2 v4l-utils vulkan-icd-loader` |
| WineASIO | This package is good for if you plan on running Ableton or FL Studio in Wine<br>1. `paru -S wineasio`<br>2. `sudo usermod -aG realtime $(whoami)`<br>For 32bit, run: `regsvr32 /usr/lib32/wine/i386-windows/wineasio.dll`<br>For 64bit, run: `wine64 regsvr32 /usr/lib/wine/x86_64-windows/wineasio.dll` |

| Miscellaneous | Commands |
| ------------- | -------- |
| Thunderbird | `sudo pacman -S thunderbird` |
| qBittorrent | `sudo pacman -S qbittorrent` |
| LF File Manager | `sudo pacman -S lf` |
| OpenRGB | 1. `paru -S openrgb`<br>2. `sudo pacman -S i2c-tools` |
| Inochi2D Session | [Download on Github](https://inochi2d.com/) |
| Rofi | `sudo pacman -S rofi` |
| Zsh plugins | 1. `sudo pacman -S zsh-syntax-highlighting zsh-autosuggestions`<br> 2. `echo "source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> .zshrc` and `echo "source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh" >> .zshrc` |
| Syncthing | `sudo pacman -S syncthing` |

| Programming | Commands |
| ------------ | -------- |
| Python | `sudo pacman -S python python-pip` |
| NodeJS | `sudo pacman -S nodejs-lts-hydrogen npm` |
| Zulu Java8 | `paru -S zulu-8-bin` |
| Zulu Java17 | `paru -S zulu-17-bin` |
| VS Code | 1. `paru -S visual-studios-code-bin`<br>2. `sudo pacman -S dotnet-runtime dotnet-sdk mono-msbuild mono-msbuild-sdkresolver mono` |

| Production | Commands |
| ---------- | -------- |
| Reaper DAW | `sudo pacman -S reaper` |
| Polyphone | `sudo pacman -S polyphone` |
| Audacity | `sudo pacman -S audacity` |
| Moonscraper Chart Editor | [Download on Github](https://github.com/FireFox2000000/Moonscraper-Chart-Editor) |
| Blender 2.79b | `paru -S blender-2.7` |
| Blender Latest | `sudo pacman -S blender` |
| Unreal Engine | Figure it out yourself |
| Inochi2D Creator | [Download on Github](https://inochi2d.com/) |
| OBS Studio Tytan652 | 1. `paru -s obs-studio-tytan652`<br>2. `sudo pacman -S v4l2loopback-dkms` |
| Kame-Editor | `paru -S kame-editor-git` |

| Communication | Commands |
| ------------- | -------- |
| Discord (Stable) | 1. `sudo pacman -S discord`<br>2. `echo ""SKIP_HOST_UPDATE": true" >> /.config/discord/settings.json` |
| Discord (Canary) | 1. `paru -S discord-canary`<br>2. `echo ""SKIP_HOST_UPDATE": true" >> /.config/discord/settings.json` |
| Skype | `paru -S skypeforlinux-stable-bin` |

| Joke Packages | Commands |
| ------------- | -------- |
| cMatrix | `sudo pacman -S cmatrix` |
| cowsay | `sudo pacman -S cowsay` |
| lolcat | `sudo pacman -S lolcat` |
| Neofetch | `sudo pacman -S neofetch` |
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
| Guitar Configurator | 1. [Download on Github](https://github.com/sanjay900/guitar-configurator)<br>2. Go to https://sanjay900.github.io/guitar-configurator/guides/linux.html to setup the udev rules for this tool. |
| Fusée Launcher Interfacée | `paru -S fusee-interfacee-tk-bin` |
| OSCDL | [Download on Github](https://github.com/dhtdht020/osc-dl) |
| WiiUDownloader | [Download on Github](https://github.com/Xpl0itU/WiiUDownloader) |
