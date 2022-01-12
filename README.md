# Archlinux installation with LUKS disk encryption and Grub bootloader

## Introduction

This installation guide is suitable for older laptops and desktops; my daily driver laptop is Intel Core 2 Duo T8100 at 2.100GHz (launch date Q1 2008)
[link](https://www.intel.com/content/www/us/en/products/sku/33916/intel-core2-duo-processor-t8100-3m-cache-2-10-ghz-800-mhz-fsb/specifications.html.) 

If you use anything better than my laptop processor, you will have a better experience.

Below is my Application recommendation after you have completed the installation guide (You can find them in official repository or AUR):

- **OS:** Archlinux using linux-zen kernel
- **Network Manager:** systemd-networkd, iwd
- **Package Helper:** yay
- **Text Editor:** vim
- **Window Manager:** sway
- **Application Launcher:** wofi
- **Status Bar:** waybar
- **Terminal:** foot
- **Browser:** qutebrowser
- **Password Manager:** bitwarden
- **Office Suite:** libreoffice, masterpdfeditor4
- **Email:** sylpheed
- **Communication:** whatsapp-nativefier
- **Music:** spotify-qt and spotifyd
- **Movie and TV Shows:** plexmediaplayer, mpv
- **Gaming:** steam
- **GUI Folder Management:** pcmanfm-gtk
- **Optimise laptop battery:** auto-cpufreq
- **Use trash-put instead of rm:** trash-cli
- **Check disk utilization quickly:** ncdu

If you have not been exposed to any other Linux distribution before, Archlinux is not a suitable first distribution for you, to quote Archwiki:

> It (Archlinux) is targeted at the proficient GNU/Linux user or anyone with a do-it-yourself attitude who is willing to read the documentation and solve their problems."

## Archlinux Installation Guide

1. Download the Archlinux ISO here; pick the server in your country or close to your country. [Link to the Archwiki download page](https://archlinux.org/download/)

	Pay attention to the file type; you want the file that ends with iso, for example:

	> archlinux-20XX.XX.XX-x86_64.iso

2. Find a thumb drive that could fit the ISO on Windows machine download and launch Rufus. [Link to download Rufus](https://rufus.ie/en/)

3. With the thumb drive loaded with bootable Archlinux iso, plug into your desired endpoint and boot into the live environment, you will see a console that is ready to receive your input.

4. Delete the hard disk partition using cfdisk

`cfdisk /dev/sda`

- Remove all partitions on the disk **This will destroy all data on the disk!**.
- Create first fat32 partition for the boot, 1 GB (This is probably overkilling it, but I want to be comfortable and future-proofing)
	> /dev/sda1
- Format the remainder of the disk to Ext4. It could take a while on slower hardware.
	> /dev/sda2

5. Format the boot partition (/dev/sda1) to fat32

`mkfs.fat -F32 /dev/sda1`

6. Now, we will encrypt the home directory using LUKS.

`cryptsetup -v --use-random luksFormat /dev/sda2`

It will now prompt you for a passphrase. **Please note that your data is gone if you lose this passphrase**, so keep it somewhere safe.

7. Once encryption is complete, we must decrypt them to continue installing.

`cryptsetup luksOpen /dev/sda2 cryptroot`

8. Format the LUKS encrypted partition to ext4, it took 15 minutes on my laptop:

`mkfs.ext4 /dev/mapper/cryptroot`

9. Both your boot partition (/dev/sda1) and (/dev/sda2) is ready to mount

```
mount /dev/mapper/cryptroot /mnt
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
```

10. Now, you will need an internet connection to your machine; the simplest way is to plug in a LAN connection to your laptop or desktop LAN port.

11. Package manager (pacman) will now pull the necessary packages and start installing.

`pacstrap /mnt base linux-zen linux-firmware vim intel-ucode sudo iwd`

12. The following command will tell the system where to boot.

`genfstab -pU /mnt >> /mnt/etc/fstab`

13. Jump into your newly installed system as root without password.

`arch-chroot /mnt /bin/bash`

14. Set the system clock to your local timezone

`timedatectl set-timezone Asia/Singapore`

15. Set the hardware clock

`hwclock --systohc --utc`

16. Set the machine hostname

`echo arch > /etc/hostname`

17. Set the language

`vim /etc/locale.gen`

- Uncomment the following line in the file

	> en_US.UTF-8 UTF-8

- Run the following command to generate locale

	`locale-gen`

- Add the following configuration parameters in the locale.conf file

	```
	echo LANG=en_US.UTF-8 > /etc/locale.conf
	echo LANGUAGE=en_US >> /etc/locale.conf
	echo LC_ALL=C >> /etc/locale.conf
	```

18. Set the root password

`passwd`

19. Tell the kernel how to boot

`vim /etc/mkinitcpio.conf`

- Under the Module section, ensure ext4 is defined, for example:

	> \# vim:set ft=sh <br>
	> \# MODULES <br>
	> \# The following modules are loaded before any boot hooks are <br>
	> \# run. Advanced users may wish to specify all system modules <br>
	> \# in this array. For instance: <br>
	> \#     MODULES=(piix ide_disk reiserfs) <br>
	> MODULES=(ext4)

- Under Hooks, ensure `encrypt` is added before `filesystem`.

	> HOOKS=(base udev autodetect modconf block encrypt filesystems keyboard fsck)

- Run the following to start building.

	`mkinitcpio -p linux`

20. Install the grub bootloader

```
pacman -S grub
pacman -S efibootmgr
grub-install --target=x86_64-efi --efi-directory=/mnt/boot/ --bootloader-id=GRUB
```

21. Configure the bootloader:

`vim /etc/default/grub`

- Edit the following section similar to the following:

	> GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:cryptroot"

- Generate the grub configuration

	`grub-mkconfig -o /boot/grub/grub.cfg`

22. Tidying up

- Exit your system
	
	`exit`

- Unmount all drive

	`umount -R /mnt`

23. Unplug the USB thumbdrive and reboot the system. You should arrive at the prompt asking you to key in credential:

> root
> password

24. If you see the command prompt waiting for instruction after you login, Archlinux installation is successful and complete.

25. **root** user should not be use, please set up normal user account now.

`useradd -m -g users -G wheel,video jackson`

26. Set the password for user:

`passwd jackson`

27. Allow user to run command as super user, uncomment the following line:

> \#%wheel ALL=(ALL) ALL

28. Exit the system, and login as user.

`exit`

29. Test the sudo function to ensure it work:

`sudo pacman -Syu`

30. Disable the root account:

`sudo passwd -l root`

---

## Recommended resources:

[Arch Wiki](wiki.archlinux.org) - Unequivocally the best wiki for Linux, search here first before hitting Duckduckgo.

This guide is heavily inspired from:

[jherrlin](https://jherrlin.github.io/posts/arch-install/)
