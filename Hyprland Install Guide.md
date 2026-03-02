# Hyprland Install Guide
If you are wanting to install Hyprland onto your system with every package required to have a working install, then follow the this guide and then return back to the main guide once it's installed.

Please note that this guide is meant to be for experienced users and should not be recommended if you don't know how advanced systemd services work. Always read the official Hyprland Wiki and Arch Wiki before using this guide.

This guide covers both install and configuration.

## Installation:
1. Run `sudo pacman -S hyprland hyprcursor hyprlock hypridle hyprpicker hyprsunset hyprpwcenter hyprsysteminfo hyprshutdown dunst polkit-kde-agent waybar swww cliphist pcmanfm-qt grim slurp alacritty qt5-wayland qt6-wayland xdg-desktop-portal-hyprland network-manager-applet` to install most of the packages necessary for a working Hyprland desktop.
2. With Paru, run `paru -S tofi waypaper` to install the rest of the packages not available in the main arch repository.
3. Go back to the [Arch Install Guide](https://github.com/Luca06Luwa/Linux-Guides/blob/WIP-md-version/Arch%20Linux%20UEFI%20Install%20Guide.md#part-3-installing-and-enabling-a-display-manager) and install uwsm as the display manager/statup.

## Configuration:
Basic configuration so that hyprland works without any issues that would require a seperate desktop to fix.

### Setting the correct apps:
Packages like the file manager and terminal aren't going to be set correctly. This changes it to match the packages that were installed in the previous step.

1. To edit the configuration file, run `nano /.config/hypr/hyprland.conf` and scroll to the my programs section.
2. Change the launch options for keybinds to match below.
```
# Set programs that you use
$terminal = alacritty
$fileManager = pcmanfm-qt
$menu = tofi-drun --drun-launch=true
```
3. Change the autostart options to match below.
```
# Autostart necessary processes (like notifications daemons, wallpaper managers, etc.)
# Or execute your favorite apps at launch like this:

exec-once = $terminal
exec-once = nm-applet
exec-once = waypaper --restore
```


### Nvidia patch: (UNOFFICIAL)
Whilst Nvidia GPU support is not official for hyprland, there is a stable workaround that the devs recommend using if you only have Nvidia.

Most of the steps to enable the functionality in the drivers have already been enabled and configured when installed. All that is needed is to tell Hyprland that it can use Nvidia drivers to render the desktop.

1. To set the environment variables to boot on Nvidia, run `nano /.cofig/hypr/hyprland.conf` and scroll to the environment variable section.
2. Add the variables below to enable Nvidia support.
```
env = LIBVA_DRIVER_NAME,nvidia
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = NVD_BACKEND,direct
```


### Hyprlock configuration:
Hyprlock does not create a configuration file, so you will have to download an [example configuration](https://github.com/hyprwm/hyprlock/blob/main/assets/example.conf).

The config file goes into the `/.config/hypr` directory.

More info can be found in the [Arch Wiki](https://wiki.archlinux.org/title/Hyprlock), and the official [Hyprland Wiki](https://wiki.hypr.land/Hypr-Ecosystem/hyprlock/)

### Final configuration options:
1. Run `systemctl --user enable waybar.service` so that waybar starts up with Hyprland.
2. Run `nano /.profile` and add the following for uwsm startup.
```
if uwsm check may-start; then
  exec uwsm start hyprland-uwsm.desktop
fi
```

Link back to the main guide: [Arch Install Guide](https://github.com/Luca06Luwa/Linux-Guides/blob/WIP-md-version/Arch%20Linux%20UEFI%20Install%20Guide.md#21-zsh-setup-and-configuration-optional)
