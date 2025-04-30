# QEMU/KVM Setup Guide for Arch-Based Distributions.
This guide is for if you want a really good gaming Virtual Machine to get around any Kernel Level Anti-Cheat or if you just want to have a easy to destroy burner install of windows. 

This guide requires you to have two GPU's. One for the host (AMD is recommended) and one for the VM (NVIDIA is recommended).

## 1. Enabling IOMMU for CPU (Only for Intel)
This step is only necessary for Intel CPU's as AMD is already enabled if the kernel detects the setting in BIOS.

| Bootloader | Instructions |
| ---------- | ------------ |
| GRUB | a. Run `sudo nano /etc/default/grub` and add `intel_iommu=on` to the `GRUB_CMDLINE_LINUX_DEFAULT=""`.<br>Should look something like this: `GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on"`.<br>b. Once the setting has been enabled, run `grub-mkconfig -o /boot/grub/grub.cfg` to rebuild the main GRUB config file.<br>c. Once the config files have been enabled, `reboot` your system to enable the changes to the kernel. |
| Systemd-Boot | a. Run `sudo nano /boot/loader/entries/arch.conf` and add `intel_iommu=on` to the end of the options line.<br>b. Once the setting has been enabled, `reboot` your system to enable the changes to the kernel. |

Once the IOMMU has been enabled for your CPU (AMD is already done if enabled in BIOS), run `dmesg | grep -i IOMMU` and verify that you see either `Intel-Iommu: Enabled` or `AMD-Iommu: Enabled`.

(To Be rewritten...)
