# Hyprland Install Guide
If you are wanting to install Hyprland onto your system with every package required to have a working install, then follow the this guide and then return back to the main guide once it's installed.

Please note that this guide is meant to be for experienced users. Always read the official Hyprland Wiki and Arch Wiki before using this guide.

This guide covers both install and cofiguration.

## Installation:
1. Run `sudo pacman -S hyprland mako polkit-kde-agent cliphist grim slurp foot qt5-wayland qt6-wayland xdg-desktop-portal-hyprland` to install most of the packages necessary for a working Hyprland desktop.
2. With Paru, run `paru -S tofi waybar-hyprland swww waypaper-git` to install the rest of the packages.


## Nvidia fix:
Whilst Nvidia GPU's don't have official support, there is a stable workaround.

Most of the steps to enable the functionallity has already been enabled and configured. All that is needed is to tell Hyprland that it can use Nvidia.

1. To set the environment variables to boot on Nvidia, run `nano /.cofig/hypr/hyprland.conf` and scroll to the environment variable section.
2. Add the variables below to enable Nvidia support.
```
env = LIBVA_DRIVER_NAME,nvidia
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
```


## Configuration:


To make DWM start on login, continue following the main guide and set up autostart with the StartX window manager (the recommended one).

Link back to the main guide: [Arch Install Guide](https://github.com/Luca06Luwa/Linux-Guides/blob/WIP-md-version/Arch%20Linux%20UEFI%20Install%20Guide.md#part-3-installing-and-enabling-a-display-manager)
