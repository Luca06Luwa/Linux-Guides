# Arch Linux UEFI Install Guide

This guide assumes that your default language is english with a us style keyboard. This guide also assumes that you are on desktop with an AMDGPU or INTEL ARC.

"Nvidia, fuck you" - Linus Torvalds

## 0. Getting the ISO
a. Go to [archlinux.org](https://archlinux.org) and click on download.<br>
b. Download the image from the torrent.<br>
c. Get some kind of [ISO burner](https://etcher.balena.io/) for a usb and just write the ISO to the USB.<br>
Note: If you do not have [a torrent client](https://www.qbittorrent.org/), you can use the download mirrors instead. Just be sure to grab the ISO file with the date printed on it.


## 1. Basic initial Setup
a. Run the command `cat /sys/firmware/efi/fw_platform_size` to verify that you have a UEFI BIOS.<br>
b. Run `timedatectl` to ensure the date and time is accurate to your current time.


## 2. Networking
If you are going to be using ethernet instead of WI-FI, then you can skip part 1.

### Part One (Wi-Fi Setup).
a. Run `ip link` to identify what network cards are available on your desktop.<br>
b. Run `iwctl` to enter the Wi-Fi configurator (iwd).<br>
c. Once in iwd, run `device list` to list the Wi-Fi card(s) available on your desktop.<br>
d. Run `station [Your wifi card] scan` to scan the local area for your network.<br>
e. Once completed, run `station [Your wifi card] get-networks` to list the recently scanned networks.<br>
f. Once you have identified your network, run `station [your wifi card] connect "[your network]"` to connect to your network.<br>
g. Once connected, press `ctrl + d` to exit iwd.<br>
h. Run `ip link` again to verify that you're getting an ip address connection to your Wi-Fi network.

### Part Two (Testing Connection).
Run `ping archlinux.org` to test if your internet connection is working correctly.<br>
Note: You can press `ctrl + c` to stop pinging the website.


## 3. Creating the Partition Tables.
If you plan on dual booting Windows 10/11, STOP this guide is not for you. If you still want to dualboot with windows, figure it out yourself, I will not help you here.

Note: If you have a blank drive that you know is empty, then you can skip step c.

a. Run `lsblk` to see what hard drives are installed in your PC.<br>
b. If you cannot identify what drive(s) you have installed, run `hdparm -i /dev/the_disk_to_be_partitioned` to double check that you've selected the right drive.<br>
c. If you only have one drive with another OS install on it and want to perform a clean install, run `gdisk /dev/the_disk_to_be_partitioned`.
- Press `x` to enable expert mode.
- Press `z` to delete the entire contents of the drive.

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

f. write changes to disk.


## 4. Format and Mount your partitions.
a. Run `lsblk` to list the drive with it's partition table created.<br>
b. Run `mkfs.ext4 /dev/root_partition` to format the root partition and run `mount /dev/root_partition /mnt` to mount the install stick to the drive.<br>
c. Run `mkfs.ext4 /dev/home_partition` to format the home partition and run `mount --mkdir /dev/home_partition /mnt/home` to create and mount the install stick to the home partition of the drive.<br>
d. Run `mkswap /dev/swap_partition` to format the swap partition and run `swapon /dev/swap_partition` to enable swap.<br>
e. Run `mkfs.fat -F 32 /dev/efi_system_partition` to format the boot partition and run `mount --mkdir /dev/efi_system_partition /mnt/boot` to create and mount the install stick to the boot partition of the drive.


## 5. Configure Mirrorlist.
This step isn't really necessary but I would highly recommend it as it sorts the package mirrors from best to worst.

Note: Since we don't have a GUI interface for file management we must do everything through command line. Don't worry though, it's completely safe.

a. Run `pacman -Sy` to update the package database on the ISO.<br>
b. Once completed, run `pacman -S pacman-contrib` to install the necessary tools for sorting the mirrorlist.<br>
c. Run `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup` to create a backup of the current mirrorlist.<br>
d. Once the copy has been created, run `nano /etc/pacman.d/mirrorlist` to see if all the servers that are listed in the file are uncommented.<br>
e. Once exited nano, run `rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist` to sort the servers in the backup file and copy it to the main file.


## 6. Download/Installing Essential Packages.
This step is where you get to actually install your system.

The following packages that will be installed are the necessary core packages and the drivers for the install as well as some drivers for wifi cards, sound cards, and your CPU manufacturer's microcode.

To install the core components, run `pacstrap -K /mnt base base-devel linux linux-headers linux-firmware linux-firmware-marvell linux-firmware-whence man-db man-pages nano sof-firmware` and before you confirm the command either add the `intel-ucode` or `amd-ucode` to the command and install the packages.


## 7. Generating the fstab file and chrooting into the install.
This step is where you will generate the drives partition UUID as without doing so will result in a system that wont know what it's doing.

a. Run `genfstab -U /mnt >> /mnt/etc/fstab` to generate the fstab file.<br>
b. Run `arch-chroot /mnt` to gain access to your install.<br>
Congratulations, you are now in your Arch Linux install. Now you will complete the next few steps on your computer.


## 8. Localisation.
This step is necessary so that your Arch Linux install knows where you are from so that the locale and timmezones will be set accordingly. It will also set the system clock for time syncronisation.

WARNING: Anything and everything listed in this part is important and messing up when entering any of these commands will result in a dead install.

a. Run `nano /etc/locale.gen` and scroll down to your country's locale and uncomment it.<br>NOTE: If you don't know your locale then uncomment `en_US.UTF-8 UTF-8`.<br>Should look something like this.
```
#en_SG.UTF-8 UTF-8
#en_SG ISO-8859-1
en_US.UTF-8 UTF-8
#en_ZA.UTF-8 UTF-8
#en_Za ISO-8859-1
```
b. Once your locale has been uncommented, save and exit nano and run `locale-gen` to generate the locale files for your install.<br>
c. Even though you've already assigned the locale with locale.gen, you will still need to echo the locale to a specific file necessary for older programs to function correctly. To do this run `echo "LANG=[the locale you selected].UTF-8" >> /etc/locale.conf` to set the legacy locale for your install.<br>
d. This step is important and should be done either way. Run `export LANG=[the locale you selected].UTF-8`.<br>
e. You can skip this step if you have a qwerty us keyboard. If you have a keyboard other than the us qwerty layout, run `echo "KEYMAP=[your keyboard layout]" >> /etc/vconsole.conf` to set the correct keyboard scheme for your keyboard.


## 9. Timezones
This step is to set the arch linux timezone correct to the one that you are in.

a. To set the timezone, run `ls /usr/share/zoneinfo` to list the unix timezones available on Arch Linux.<br>
b. Once you have found your timezone, run `ln -sf /usr/share/zoneinfo/[Your Country Here]/[Your Timezone Here] /etc/localtime` to register a symbolic link for your timezone.<br>
c. To link the software clock to the hardware clock of your computer, run `hwclock --systohc` to set the hardware clock.


## 10. Configure Pacman/Package Manager.
This step is where you will configure pacman to be able to download multiple packages at the same time and also enable the ability to download 32-bit packages through the Multilib repository.

a. Run `nano /etc/pacman.conf` to enter the pacman config file.<br>
b. Uncomment the line that you see below.<br>

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

c. In the Misc Options area, add/uncomment the following items. `ParallelDownloads = 5`, `Color` and `ILoveCandy`.<br>
d. Once saved, run `pacman -Sy` to apply the modified changes to the config file and download the new repository.


## 11. Installing more packages and enabling system services.
This step is where you are going to install some more packages and some miscellaneous drivers for connecting the internet as well as enabling some necessary system functions.

Note: Skip the fstrim function if you don't have an SSD.

a. Run `pacman -S git networkmanager reflector pacman-contrib bash-completion` to install the listed packages.<br>
b. Once all packages have been installed, enable the following services to start the necessary drivers and system functions.
```
systemctl enable NetworkManager.service
systemctl enable fstrim.timer
systemctl enable reflector.timer
```

## 12. Hostname Configuration and User Setup.
This step is where you will set the name of the computer name and add your user account(s). 

a. Run `echo "[Insert Computer Name Here]" >> /etc/hostname` to set the name of the computer.<br>
b. Run `nano /etc/hosts` and add the following into the file.
```
127.0.0.1        localhost
::1              localhost
127.0.1.1        [Add same hostname as before.]
```

c. To setup the administrator account/root, run `passwd` to create the account and set the root password.<br>
d. To add/create a user account, run `useradd -m -G wheel,storage,power -s /bin/bash [Insert Username Here]` to create your user account.<br>
e. Run `passwd [Insert Username Here]` to set the password for the user account that you just created.<br>
f. Once the user account(s) have been setup, run `EDITOR=nano visudo` to edit the administrative permissions/sudo.<br>
Uncomment `%wheel ALL=(ALL) ALL` and add `Defaults rootpw` to the bottom of the file.

Note: The `Defaults rootpw` is so that you use the root password instead of your user password for running sudo commands which makes your install behave more like windows.


## Bootloader.
Here is where you will be able to play a little choose your own adventure story for your install. There are three commonly used boot loaders that people tend to install on their systems. Choose the one you prefer and forget about the other ones.

GRand Unified Bootloader / GRUB. (Easy Mode)<br>
This bootloader is the most common across most distro's and has the most documentation around customisation and configurations.<br>

rEFInd. (Medium Mode)<br>
This bootloader was originally designed for dualbooting OSX/mac but has been improved to support many more operating systems. This is what I would recommend if your trying to dualboot windows for some reason. (Still not helping with setting up windows dualboot).

Systemd-Boot. (Hard Mode)<br>
This bootloader is what I would recommended to you as the packages are preinstalled onto your system during install and are only targeted to booting a singular OS.

If you choose to install GRUB then ONLY do 13a.<br>
If you choose to install rEFInd then ONLY do 13b.<br>
If you choose to install Systemd-Boot then ONLY do 13c.

### 13a. GRUB. (Linux dual boot/Easy Mode)
a. Run `pacman -S grub efibootmgr` to install the necessary packages for installing GRUB.<br>
b. Once the packages are downloaded, run `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` to inject and install GRUB into your system.<br>
c. Once the bootloader is installed, run `grub-mkconfig -o /boot/grub/grub.cfg` to generate the configuration files for the bootloader.

### 13b. rEFInd. (Medium Mode)
a. Run `pacman -S refind` to install the necessary packages for installing rEFInd.<br>
b. Once the packages are downloaded, run `refind-install` to inject and install rEFInd to your system.<br>
c. Once the bootloader is installed, run `nano /boot/refind_linux.conf` and modify the "Boot with standard options" line so that it has `initrd=[Your CPU Brand]-ucode.img` at the end. (This will need fixing in the future)

### 13c. Systemd-Boot. (Requires manual entries/Hard Mode)
a. You will not need to download any packages when installing SystemD-Boot but you will need to verify the presence of the efi firmware on your system, to do this run `ls /sys/firmware/efi/efivars` to verify if the system efi firmware is mounted and installed.<br>
b. Once confirmed the presence of efi firmware, run `bootctl install` to inject and install Systemd-Boot to your system.<br>
c. Once the bootloader is installed, run `nano /boot/loader/entries/arch.conf` and add the following lines.
```
title Arch Linux
linux /vmlinuz-linux (change this depending on what kernel you have. for example, linux-lts for the lts kernel or linux-zen for the zen kernel).
initrd /initramfs-linux.img
```

d. Once added everything into the file, run `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition) rw" >> /boot/loader/entries/arch.conf` to add the partition UUID for the root partition. This is important as it tells Arch Linux to only boot to that drive. (Credit to Glorious Eggroll for this command.)


## 14. Graphics Drivers
This step is what I like to call "NIGHTMARE MODE" as you will be installing your GPU drivers. The drivers have been sorted based on what manufacturer your card is from. So select the one that matches your card.

Note: There are two NVIDIA drivers, the proprietary driver is for gtx700 series to rtx3000 series and the open modules are for rtx2000 series and newer. So PLEASE be careful when installing your GPU driver for NVIDIA. 

| Manufacturer | Instructions |
| ------------ | ------------ |
| AMD | For AMDGPU drivers, run `pacman -S xf86-video-amdgpu mesa opencl-rusticl-mesa vulkan-radeon lib32-mesa lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers. |
| INTEL | For INTEL ARC drivers, run `pacman -S xf86-video-intel mesa intel-compute-runtime intel-media-driver vulkan-intel lib32-mesa lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers. |
| NVIDIA (PROPRIETARY) | For MAXWELL to ADA LOVELACE cards, run `pacman -S nvidia-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers. |
| NVIDIA (Open GPU Kernel Modules) | For all newer cards from TURING onwards, run `pacman -S nvidia-open-dkms nvidia-utils libglvnd opencl-nvidia lib32-nvidia-utils lib32-libglvnd lib32-opencl-nvidia nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader` to install the drivers. |


## Configure Drivers for KMS/Wayland Support.
This step should only be done with the version that matches your card.

If you have installed the NVIDIA drivers then ONLY do step 14a.<br>
If you have installed the AMDGPU drivers then ONLY do step 14b.<br>
If you have installed the INTEL ARC drivers then ONLY do step 14c.

WARNING: If for whatever reason you mess up on these steps your system is dead!

### 15a. NVIDIA
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)`<br>
b. Once those modules have been added, save the file and run `mkinitcpio -P` to regenerate the kernel initramfs.<br>

### 15b. AMDGPU
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... amdgpu ...)`<br>
b. Once this module has been added, save the file and run `mkinitcpio -P` to regenerate the kernel initramfs.<br>

### 15c. INTEL
a. Run `nano /etc/mkinitcpio.conf` and edit the `MODULES()` line to look like this.<br>
`MODULES(... i915 ...)`<br>
b. Once this module has been added, save the file and run `mkinitcpio -P` to regenerate the kernel initramfs.<br>


## 16. Unmount drives and Reboot system.
a. Once you have finished setting up drivers, run the `exit` command to return back to the install drive.<br>
b. Once you're back in the install drive, run `umount -r /mnt` to safely unount the install partitions from the install drive.<br>
c. `reboot`<br>
Congratulations. You have sucessfully installed the base version of Arch Linux.

However you're not done just yet. You still need to install a desktop and setup audio drivers.


## 17. General First Install Checks.
This step is just a general after installation check to ensure that nothing went wrong with the install. This step also contains configuration guides for regenerating mirrorlists automatically.

Important: Now that you're actually using your Arch Linux install now, you will need to use the sudo command in order to perform root/administrator privilages.

a. Once rebooted and logged in, run `systemctl --failed` to verify that a sucessful boot had occured.<br>
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


## 18. Enabling AUR support and flatpak.
This step enables the ability to use the best part of Arch linux. The Arch User Repository (AUR). This will also install flatpak for official universal packages (this is simialr to what windows does).

Traditionally, packages from the AUR have to be downloaded and compiled onto your system from source code, but with a program called an AUR Helper, it builds and installs everything for you.

a. To install an AUR helper, run `git clone https://aur.archlinux.org/paru-bin.git` to download the required files to compile Paru (the AUR helper).<br>
b. Once downloaded, `cd paru-bin` to go into the newly downloaded folder.<br>
c. Once you are in the `paru-bin` folder, run `makepkg -si` to install Paru.<br>
d. Once Paru is installed you update all packages installed on your computer through it as it also acts as a pacman replacement.<br>
e. Now that Paru is installed, you can now install flatpak by running `sudo pacman -S flatpak` to install the main flatpak app and repository.<br>
f. Normally, once flatpak is installed you would run `reboot` to complete the installation of flatpak. But we'll do that later.


## 19. Audio Drivers.
This step is required if you want to have a working audio and video sharing setup. 

Note: One of the packages, `pipewire` to be exact, is a requirement for Wayland since by itself Wayland does NOT allow screen capture for programs.

To install audio drivers, run `sudo pacman -S alsa-ucm-conf alsa-utils alsa-plugins pavucontrol pipewire pipewire-audio pipewire-alsa pipewire-jack pipewire-pulse lib32-pipewire lib32-pipewire-jack qpwgraph wireplumber` to install all the packages needed for a working audio setup.


## 20. Graphical Environment.
This step is probably the most confusing to new users. (It is also the most difficult part for me to mantain as stuff changes every few months).

Currently, there are two well known video drivers that a linux system can have installed, both of them being Wayland and Xorg (legacy). This guide is mainly focused on Xorg as most apps still use it (like most games), however if you want to use Wayland instead of Xorg then it's already been set up and enabled.

If you do not want to use Xorg at all and want to have a pure Wayland configuration, then you can skip part 1 of this step and just install a Wayland based desktop environment.

Note: Most Wayland compositors may not work with Nvidia GPU's, so if you have Nvidia GPU use Xorg.

### Part 1. Installing Xorg.
To install Xorg and all it's necessary packages, run `sudo pacman -S xorg xorg-xinit` to install Xorg.

### Part 2. Selecting your Desktop Environment and or Window Manager.
Note: Most desktops are now based on Wayland and have Xorg as fallback.

| Xorg Desktop Environment | Instructions |
| ------------------------ | ------------ |
| AwesomeWM | Run `sudo pacman -S awesome alacritty pcmanfm-qt` to install the packages for a working install of AwesomeWM. |
| DWM | DWM is the most barebones Window manager, as a result of this the instuctions have benn moved. Go to [DWM Install Guide](https://github.com/Luca06Luwa/Linux-Guides/blob/WIP-md-version/DWM%20Install%20Guide.md) if you want to install DWM. |
| i3 | Run `sudo pacman -S i3 alacritty pcmanfm-qt dmenu` to install the packages for a working install of i3. |
| Xfce | Run `sudo pacman -S xfce xfce-goodies network-manager-applet` to install the packages for a working install of Xfce. |

| Wayland Desktop Environments | Instructions |
| ---------------------------- | ------------ |
| Gnome | Run `sudo pacman -S gnome gnome-tweaks xdg-desktop-portal-gnome` to install the packages for a working install of Gnome. |
| Hyprland | Because Hyprland has many first party dependencies, visit the [Hyprland wiki](https://wiki.hyprland.org/) to have a properly working install. |
| KDE Plasma | Run `sudo pacman -S plasma kde-applications qt5-wayland xdg-desktop-portal-kde` to install the packages for a working install of KDE Plasma. When prompted, select the VLC backend for audio. |
| Sway | Note: If you have an existing i3 installation, this will be a drop in replacement as sway uses the same i3 config files.<br>Run `sudo pacman -S sway swaylock swayidle swaybg waybar mako polkit-kde-agent qt5-wayland qt6-wayland cliplist light grim slurp foot xdg-desktop-portal-wlr` to install most of the packages reqired for a working install of Sway.<br>With Paru, run `paru -S tofi` to install the application launcher. |

### Part 3. Installing and enabling a display manager.
Most display managers are designed to work with the desktop that they are typically packaged with. The only display managers that work universally are StartX, LightDM, wlroots on TTY, and uwsm. 

| Display Manager | Instructions |
| --------------- | ------------ |
| GDM (Gnome Only) | Note: Since GDM is included with Gnome you don't need to install anything.<br>To enable the Display Manager upon reboot run `sudo systemctl enable gdm.service`. |
| SDDM (X11 & Wayland (except sway)) | Run `sudo pacman -S sddm` to install SDDM and then run `sudo systemctl enable sddm.service` to enable the Display Manager upon reboot. |
| LightDM (X11 Only) | Run `sudo pacman -S lightdm` to install the base version of lightDM and run `sudo systemctl enable lightdm.service`  to enable the Display Manager upon reboot.<br>Since LightDM does not include a environment to run on you wil have to install one of the greeters listed below. |
| StartX (X11 Only) | Since StartX is kind of difficult to setup, I will simply link you to the [Arch Wiki](https://wiki.archlinux.org/title/Xinit#Autostart_X_at_login) for instructions. |
| wlroots on TTY (Wayland Only) | Since most wayland compositors are based on wlroots, they do not allow launching with a Display Manager. So, I will simply link to the [Arch Wiki](https://wiki.archlinux.org/title/Sway#Automatically_on_TTY_login) for instructions on how to setup TTY login. |
| uwsm (Wayland Only) | 1. Run `sudo pacman -S uwsm` to install uwsm<br>2. Follow the [Arch Wiki](https://wiki.archlinux.org/title/Universal_Wayland_Session_Manager) |

### (Only for LightDM) Part 4. Choose the greeter you want to use for LightDM.
If you have installed any other display manager other than LightDM, then you can skip this step.

By itself, LightDM does not come with any user interfaces. There are two versions that are listed here, one uses GTK as a customiser which is good if you have installed Xfce and the other uses Webkit2 as a customiser. If you want a style that looks like Gnome/Xfce, then install the GTK version. If you want a style that's easy to configure and looks great, then install the Webkit2 version.

| Greeter | Instructions |
| ------- | ------------ |
| GTK | Run `sudo pacman -S lightdm-gtk-greeter lightdm-gtk-greeter-settings` to install the GTK greeter and configurator tool. |
| Webkit2 | Run `sudo pacman -S lightdm-webkit2-greeter` to install the webkit2 greeter. |


## 21. Zsh Setup and Configuration. (Optional)
This step is if you want to use a different terminal shell from the default bash shell.

Note: Zsh is plugin based for most customisations and most of those plugins are easy to install, as a result, most plugin managers such as Oh My Zsh are not needed and are actually considered bloatware.

a. To install Zsh, run `sudo pacman -S zsh zsh-completions` to install core Zsh and some first party plugins.<br>
b. Once Zsh is installed, run `zsh` to begin the initial setup.<br>
c. Now that Zsh is configured to your liking, run `chsh -s /usr/bin/zsh` to set Zsh as your default terminal shell.

Tip: You might want to move some code from the `.bashrc` file to the `.zshrc` file (e.g. the prompt and the aliases). It's also recommended to move code from the `.bash_profile` file to the `.zprofile` file (e.g. the code that makes your window manager work).


## 22. Gstreamer Full Support. (Everything except KDE and Window Managers)
This step only applies to users who have installed a Desktop Environment/Window manager and don't want to utilise VLC for audio backend. Users who have installed a Window Manager or have installed KDE with VLC as a backend can skip this step entirely.

To install Gstreamer, run `sudo pacman -S gstreamer lib32-gstreamer gst-libav gst-plugins-bad gst-plugins-base gst-plugins-good gst-plugins-ugly gst-plugins-pipewire gstreamer-vaapi` and `paru -S gst-plugin-libde265 gst-plugins-openh264` to install the base package and other audio codec's.


## 23. Reboot and login.
Remember how I said at step 18 that we would skip the reboot part for flatpak, guess what, it's here. 

Now that you have everything installed, `reboot` and login to your user account and then you should see the Desktop you installed.<br>
Congratulations You have sucessfully installed Arch Linux.


## Applications.
This is a list of all programs that have linux support that I am aware of. There are games and other programs in here too.

Note 1: This list is only if your using the terminal for installing packages and before you install a program, always remember to run a `sudo pacman -Sy` or `sudo pacman -Syu` every few months to make sure that the repos and packages are up to date so that there is no incompatibility issues.

Note 2: If a program is distributed as an appimage, please use AppImageLauncher to install it instead of running it manually. It's the same thing as a windows exe that you aren't sure about.

This list has been seperated into multiple sections based on what the package relates to.

| Essential Packages | Commands |
| ------------------ | -------- |
| AppImageLauncher | `paru -S appimagelauncher` |
| 7-Zip | `sudo pacman -S 7zip` |
| Windows 11 Fonts | `paru -S ttf-ms-win11-auto` |
| Timeshift | `sudo pacman -S timeshift` |
| Downgrade | `paru -S downgrade` |
| Brave Browser | `paru -Sy brave-bin` |
| Bluetooth | 1. `sudo pacman -S bluez bluez-utils`<br>2. `sudo systemctl enable bluetooth.service` |

| Game Launchers | Commands |
| -------------- | -------- |
| Steam | `sudo pacman -S steam` |
| Lutris | `sudo pacman -S lutris`<br>Note: Lutris requires you to have already installed the base version of Wine |
| YARG | 2. `sudo pacman -S hidapi systemd-libs`<br>2. [Download on Github](https://github.com/YARC-Official/YARC-Launcher)|
| Heroic Games Launcher | `paru -S heroic-games-launcher-bin` |

| Minecraft Launchers | Commands |
| ------------------- | -------- |
| Minecraft (Official) | `paru -S minecraft-launcher`<br>Note: Minecraft requires java 21 lts for builds from 1.21 onwards and java 8 lts can be used for any builds from classic to 1.12. |
| Prism Launcher | `sudo pacman -S prismlauncher` |
| Modrinth Launcher | [Download from website](https://modrinth.com/app) |

| Games | Commands |
| ----- | -------- |
| osu! | `paru -S osu-laser-bin` |
| Katawa Shoujo | `paru -S katawa-shoujo` |
| Clone Hero v1.0.0.4080-final | `paru -S clonehero` |
| Clone Hero v1.1.0.4261-PTB | `paru -S clonehero-ptb` |
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
| CEMU (Upstream) | `flatpak install cemu` |
| mGBA (Arch Package) | `sudo pacman -S mgba-qt` |
| Snes9x (Arch Package) | `sudo pacman -S snes9x-gtk` |
| Lime3DS (Upstream) | `flatpak install lime3ds` |
| ñ (PabloMK7 Fork) | Lost media |
| Azahar (Upstream) | `flatpak install azahar` |

| Media | Commands |
| ----- | -------- |
| Ani-Cli | `paru -S ani-cli` |
| MPV | `sudo pacman -S mpv` |
| VLC | `sudo pacman -S vlc` |
| VLC-luajit | `paru -S vlc-luajit`<br>Note: This one is useful if you plan to use oobs-tytan652 |
| GoXLR-Utility | `paru -S goxlr-utility` |
| Physical Media | `sudo pacman -S libcdio libdvdread libdvdcss libdvdnav libbluray libaacs`<br>Note: If your using KDE applications to read cd's, run `sudo pacman -S audiocd-kio` to install the package. |

| Compatibility Tools/Wine | Commands |
| ------------------------ | -------- |
| Proton-GE | [Download on Github](https://github.com/GloriousEggroll/proton-ge-custom) |
| Wine-GE | [Download on Github](https://github.com/GloriousEggroll/wine-ge-custom) |
| GameMode | `sudo pacman -S gamemode lib32-gamemode` |
| Protonup-QT | `paru -S protonup-qt` |
| Wine | Please note that wine is literally a dependency nightmare if you don't know what you are doing.<br>1. `sudo pacman -S wine-staging winetricks`<br>2. `sudo pacman -S --needed alsa-lib alsa-plugins cups dosbox ffmpeg giflib gnutls gst-plugins-base-libs gtk3 lib32-alsa-lib lib32-alsa-plugins lib32-giflib lib32-gnutls lib32-gst-plugins-base-libs lib32-gtk3 lib32-libpulse lib32-libva lib32-libxcomposite lib32-libxinerama lib32-ocl-icd lib32-sdl2-compact lib32-v4l-utils lib32-vulkan-icd-loader libgphoto2 libpulse libva libxcomposite libxinerama ocl-icd samba sane sdl2-compact v4l-utils vulkan-icd-loader wine-gecko wine-mono libpng lib32-libpng libldap lib32-libldap mpg123 lib32-mpg123 openal lib32-openal libjpeg-turbo lib32-libjpeg-turbo ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt` |
| WineASIO | This package is good for if you plan on running Ableton or FL Studio in Wine<br>1. `paru -S wineasio`<br>2. `sudo usermod -aG realtime $(whoami)`<br>3. Run `wine64 regsvr32 /usr/lib/wine/x86_64-windows/wineasio64.dll` and `regsvr32 /usr/lib/wine/i386-windows/wineasio32.dll` to register the 32bit and 64 bit registry |

| Miscellaneous | Commands |
| ------------- | -------- |
| qBittorrent | `sudo pacman -S qbittorrent` |
| OpenRGB | 1. `sudo pacman -S openrgb`<br>2. `sudo pacman -S i2c-tools` |
| Inochi2D Session | [Download on Github](https://inochi2d.com/) |
| Rofi | `sudo pacman -S rofi` |
| Zsh plugins | 1. `sudo pacman -S zsh-syntax-highlighting zsh-autosuggestions`<br> 2. `echo "source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> .zshrc` and `echo "source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh" >> .zshrc` |
| Syncthing | `sudo pacman -S syncthing` |
| Bridge/Enchor.us | [Download on Github](https://github.com/Geomitron/Bridge) |

| Programming | Commands |
| ----------- | -------- |
| Python | `sudo pacman -S python python-pip` |
| NodeJS | `sudo pacman -S nodejs-lts-jod npm` |
| Zulu Java8 | `paru -S zulu-8-bin` |
| Zulu Java17 | `paru -S zulu-17-bin` |
| Zulu Java21 | `paru -S zulu-21-bin` |
| VS Code | 1. `paru -S visual-studios-code-bin`<br>2. `sudo pacman -S dotnet-runtime dotnet-sdk aspnet-runtime mono-msbuild mono-msbuild-sdkresolver mono` |

| Production | Commands |
| ---------- | -------- |
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
| HyFetch (Updated Neofetch fork) | Note: This fork requires you to replace the neofetch command with neowofetch to use the updated fork.<br>`sudo pacman -S hyfetch` |
| Fastfetch | Note: I would recommend you use this fork as it works better and is more feature complete.<br>`sudo pacman -S fastfetch` |
| Activate Linux | `paru -S activate-linux-git` |
| Arch Linux Wallpapers | This isn't a joke package. It's literally just some Arch Linux themed wallpapers.<br>`sudo pacman -S archlinux-wallpaper` |

| Flatpak Packages | Commands |
| ---------------- | -------- |
| Flatseal | `flatpak install flatseal` |
| OBS Studio | 1. `flatpak install obs-studios`<br>2. `sudo pacman -S v4l2loopback-dkms` |
| Discord | `flatpak install discord` |
| Extension Manager | This package is ONLY for Gnome.<br>`flatpak install ExtensionManager` |
| Bottles | `flatpak install bottles`<br>Note: Bottles requires you to have already installed the base version of Wine |
| Floorp Browser | flatpak install floorp |

| System Diagnostic Tools | Commands |
| ----------------------- | -------- |
| Mangohud | `sudo pacman -S mangohud lib32-mangohud` |
| GOverlay | `sudo pacman -S -S goverlay` |
| Btop++ | `sudo pacman -S btop` |
| Htop | `sudo pacman -S htop` |

| Device Hacking | Commands |
| -------------- | -------- |
| Wireshark | `sudo pacman -S wireshark-qt` |
| Santroller Configurator | [Download on Github](https://github.com/Santroller/Santroller) |
| Fusée Launcher Interfacée | `paru -S fusee-interfacee-tk-bin` |
| OSCDL | [Download on Github](https://github.com/dhtdht020/osc-dl) |
| WiiUDownloader | [Download on Github](https://github.com/Xpl0itU/WiiUDownloader) |
