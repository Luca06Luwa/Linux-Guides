# DWM Install Guide
If you are wanting to install DWM onto your system, then follow the this guide and then return back to the main guide once it's installed.

There are two ways of installing DWM on Arch Linux:

### Upstream:
The first being the upstream method in which you download the package from the source tree and manually compiling the package. Keep in mind you will not have pacman keeping track of outdated version if installed this way.<br>If you wish to install manually, go to the Manual Guide.

### Arch User Repository:
The other method being you install from the Arch User Repository and building from there. Pacman will keep track of updates and versions.<br>If you wish to install through the AUR, go to the Arch User Repository Guide.

Whichever method you choose will give you the same result, that being you will have a functioning DWM install.

## Manually:

Note: You are responsible for ensuring that dwm is up to date after every stable release as pacman will not help you here.

1. Run `sudo pacman -S freetype2 libx11 libxft libxinerama dmenu alacritty pcmanfm-qt` to install the dependenciesneed for building and having a working desktop.<br>
2. Run `git clone https://git.suckless.org/dwm`to download the source files.<br>
3. Run `sudo make clean install` to install dwm base.<br>
4. Edit the `config.h` file to your liking and ensure the required packages are assigned.<br>
5. Rebuild the package with the same build command.

To make DWM start on login, continue following the main guide and set up autostart with the StartX window manager (the recommended one).

## Arch User Repository:

1. Run `sudo pacman -S dmenu alacritty pcmanfm-qt` to install the dependencies needed to have a working desktop.<br>
2. Run `git clone https://aur.archlinux.org/dwm.git`to download the source files from the AUR.<br>
3. Edit the `config.h` file to your liking and ensure the required packages are assigned.<br>
4. Run `makepkg -si` to compile DWM and apply the customisations.

To make DWM start on login, continue following the main guide and set up autostart with the StartX window manager (the recommended one).

Link back to the main guide: [Arch Install Guide](https://github.com/Luca06Luwa/Linux-Guides/blob/WIP-md-version/Arch%20Linux%20UEFI%20Install%20Guide.md#part-3-installing-and-enabling-a-display-manager)
