# QEMU/KVM Setup Guide for Arch-Based Distributions.
This guide is for if you want a really good gaming Virtual Machine to get around any Kernel Level Anti-Cheat or if you just want to have a easy to destroy burner install of windows. 

This guide requires you to have two GPU's. One for the host (AMD is recommended) and one for the VM (NVIDIA is recommended). This guide is also not going to have any instructions for rEFInd bootloader.

## 1. Enabling IOMMU for CPU (Only for Intel)
This step is only necessary for Intel CPU's as AMD is already enabled if the kernel detects the setting in BIOS.

| Bootloader | Instructions |
| ---------- | ------------ |
| GRUB | a. Run `sudo nano /etc/default/grub` and add `intel_iommu=on` to the `GRUB_CMDLINE_LINUX_DEFAULT=""`.<br>Should look something like this: `GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on"`.<br>b. Once the setting has been enabled, run `grub-mkconfig -o /boot/grub/grub.cfg` to rebuild the main GRUB config file.<br>c. Once the config files have been enabled, `reboot` your system to enable the changes to the kernel. |
| Systemd-Boot | a. Run `sudo nano /boot/loader/entries/arch.conf` and add `intel_iommu=on` to the end of the options line.<br>b. Once the setting has been enabled, `reboot` your system to enable the changes to the kernel. |

Once the IOMMU has been enabled for your CPU (AMD is already done if enabled in BIOS), run `dmesg | grep -i IOMMU` and verify that you see either `Intel-Iommu: Enabled` or `AMD-Iommu: Enabled`.


## 2. Verifying IOMMU groups and saving group ids.
This step is where you will identify the IOMMU groups allocated to your GPU's and saving them for use later.

a. Paste this command and make sure to make sure your terminal does not change anything, just paste it in and that's it.
```
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
b. Save your GPU iommu groups for use in device isolation later on.<br>
Example: (This example was pulled from the Arch wiki).
```
IOMMU Group 13:
	 06:00.0 VGA compatible controller: NVIDIA Corporation GM204 [GeForce GTX 970] [10de:13c2] (rev a1)
	 06:00.1 Audio device: NVIDIA Corporation GM204 High Definition Audio Controller [10de:0fbb] (rev a1)
```
c. Save your GPU's IOMMU group ids in a notepad. The group ids are the ones similar to the `[10de:13c2]` line.<br>
Note: If your GPU is assigned in a group with another device, you're going to need to install the zen kernel and use that for virtualization due to specific patches.


(To Be rewritten...)
