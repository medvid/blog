---
title: "Prepare multi-platform exherbo build environment with qemu"
date: 2017-09-03T20:41:41+03:00
draft: true
---

* Download QEMU and add to PATH: https://qemu.weilnetz.de/w64/
* Create qemu raw disk image:

	    qemu-img create -f raw exherbo.img 30G
	   
* Mount the image as loopback device:

	    losetup /dev/loop0 exherbo.img

* Create filesystem partitions:

	    fdisk /dev/loop0
	    n
	    p
	    1
	    ENTER
	    +2G
	    n
	    p
	    2
	    ENTER
	    ENTER
	    t
	    1
	    82
	    w
    
* Probe the updated partition layout:

	    partx -u /dev/loop0
    
* Create the filesystems:

		mkswap /dev/loop0p1
		mkfs.ext4 /dev/loop0p2

* Mount the root filesystem:

		mkdir /mnt/exherbo
		mount /dev/loop0p2 /mnt/exherbo

* Extract exherbo musl stage to the root filesystem:

	    cd /mnt/exherbo
	    wget http://somasis.com/stages/exherbo-x86_64-pc-linux-musl-current.tar.xz
	    tar xJpf exherbo-x86_64-pc-linux-musl-current.tar.xz
    
* Mount virtual filesystems:

		mount --rbind /dev dev
		mount --bind /sys sys
		mount -t proc none proc

* Chroot to the extracted rootfs:

		env -i TERM=$TERM SHEL=/bin/bash HOME=/root chroot . /bin/bash

* Load enviroment:

		source /etc/profile

* Edit paludis environment:

	* `/etc/paludis/bashrc`: replace `-march=native` with `-march=x86-64 -mtune=generic` in x86_64_pc_linux_musl_CFLAGS and CXFLAGS, and `-march=i686 -mtune=generic` in i686 flags
	* `/etc/paludis/options.conf`:  add the following lines:

			*/* build_options: symbols=strip -recommended_tests
			*/* providers: -openssl libressl
			*/* providers: -pkg-config pkgconf
			*/* -bash-completion -vim-syntax
			net-misc/wget providers: -* gnutls
		
		* `/etc/paludis/package_mask.conf`: mask linux kernels > 3.4:

			>=sys-kernel/linux-headers-3.5
			>=sys-apps/iproute2-3.5

		* `/etc/paludis/suggestions.conf`: adjust recommendations:

			sys-apps/eudev -sys-kernel/linux-headers
			dev-libs/glib -dev-libs/glib-networking

	* Edit /usr/include/bits/syscall.h and comment out __NR- and SYS_ starting from kcmp (linux >= 3.5)
	
	* Sync repositories:

		cave sync
			
	* Resolve wget with gnutls provider:

		cave resolve net-misc/wget -x1 --no-dependencies-from '*/*'

	* Uninstall openssl:

		cave uninstall openssl -x1 --uninstalls-may-break '*/*'

	* Resolve libressl:

		cave resolve libressl -x1 

	* Resolve broken packages:

		cave resolve lynx libssh2 libarchive python:2.7 python:3.6 curl git iputils openssh ca-certificates -x1 -Ca

	* Change wget provider to libressl and re-resolve it.

	* Resolve dev-util/pkgconf and virtual/pkg-config

	* Uninstall unneeded packages

		cave uninstall dev-util/pkg-config emacs gnutls glib-networking iproute2 --uninstalls-may-break glib --purge '*/*' 
		
    * Downgrade linux kernel to 3.4:

		cave resolve linux-headers --uninstalls-may-break eudev --uninstalls-may-break sydbox --permit-downgrade linux-headers -x1
	
	* Resolve musl and patch the bits/syscall.h header again

	* Resolve installed packages

		cave resolve installed-slots --without musl --without sydbox --without gcc:6 --without libstdc++:6 --without libgcc:6 --without libatomic:6 -e -x1 -Ca

TBD
			