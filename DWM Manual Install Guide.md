# DWM Manual Install Guide
If you are wanting to build DWM manually instead of installing through the Arch User Repository, then follow the steps bellow.

Note: You are responsible for ensuring that dwm is up to date after every stable release as pacman will not help you here.

1. Run `sudo pacman -S libx11 libxft libxinerama dmenu alacritty pcmanfm-qt` to install the dependencies.<br>
2. Run `git clone https://git.suckless.org/dwm`to download the source files.<br>
3. Run `sudo make clean install` to install dwm base.<br>
4. Edit the `config.h` file to your liking and ensure the required dependencies are assigned.<br>
5. Rebuild the package with the same build command.

To make DWM start on login, continue following the main guide and set up autostart with the StartX window manager (the recommended one).
