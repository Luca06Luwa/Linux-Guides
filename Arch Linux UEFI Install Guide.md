# Arch Linux UEFI Install Guide
Small note: This took soo long to rewrite this guide as there is soo much stuff in it that had to be rewritten from scratch.

This guide assumes that your default language is english.

## 1. Basic initial Setup
a. Run the command `ls /sys/firmware/efi/efivars` to check if your booted into UEFI.</br>
b. Run `timedatectl status` to ensure the date and time is accurate.

## 2. Networking
If you're not going to be using WI-FI, then skip steps b to g.

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
