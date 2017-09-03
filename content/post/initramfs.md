---
title: Running Void Linux with an old kernel
date: 2017-08-17
---

## Goal

* Replace the default kernel used in the Void Linux distribution with the legacy kernel version 3.10.

## Notes

* The older Linux kernels are not available in the official Void Linux repositories.
* Void Linux provides the kernel with dynamically loaded modules (initial ramdisk requried to load the required kernel modules for ext4 and scsi)
* Void Linux is using grub2 as bootloader and dracut as initramfs generator
* I had no experience with dracut
* Older Linux 3.x kernels cannot be compiled with newer GCC (>=5.x) without patching
* I had no desire or time to install GCC 4, configure and build custom Linux kernel with all modules compiled statically
* I found prebuilt desktop kernel for x86_64 at the openSUSE Build Service repository - https://build.opensuse.org/package/show/home:tiwai:kernel:3.10/kernel-desktop

## Instructions

* Download all RPMs from the OBS repository

        http://download.opensuse.org/repositories/home:/tiwai:/kernel:/3.10/openSUSE_13.1/
    
* Extract all rpms to the appropriate locations in /boot, /lib/modules and /usr/src
    
* Make sure ./build and ./source symbolic links in /lib/modules point to valid locations in /usr/src:

        $ cd /lib/modules/3.10.10-3.g8038aea-desktop
        $ file build
        build: symbolic linux to /usr/src/linux-3.10.10-3.g8038aea-obj/x86_64-desktop
        $ file source
        source: symbolic link to /usr/src/3.10.10-3.g8038aea
    
* Generate module dependencies (this creates files /lib/modules/3.10.10-3.g8038aea-desktop/modules.dep{,.bin})

        $ depmod 3.10.10-3.1.g8038aea-desktop

* Generate initramfs

        $ dracut -fv /boot/initramfs-3.10.10-3.g8038aea-desktop.img 3.10.10-3.g8038aea-desktop

* Generate GRUB config. Make sure both vmlinuz and initramfs were recognized.

        $ grub-mkconfig -o /boot/grub/grub.cfg
        ...
        Found linux image: /boot/vmlinuz-3.10.10-3.g8038aea-desktop
        Found initrd image: /boot/initramfs-3.10.10-3.g8038aea-desktop.img
        done

* Reboot and select "Void GNU/Linux, with Linux 3.10.10-3.g8038aea-desktop" in the advanced GRUB menu
