---
title: Installing Exherbo on VirtualBox
date: 2015-03-01
aliases:
    - /exherbo/2015/03/01/vbox/
---

* To install Exherbo system on VirtualBox, you need existing VirtualBox Linux machine with the same architecture (x86_86 or i686).
  I'm using ArchLinux for this purpose. Create Exherbo guest (exherbo-x64.vbox), create virtual harddisk (exherbo-x64.vdi), and attach it to ArchLinux guest:

TODO image

* Launch ArchLinux guest.

`/dev/sdb` drive points to exherbo-x64.vdi.

* Create new partition scheme (1GB swap, everything else for root partition):

        fdisk /dev/sdb
        n
        p
        1
        <ENTER>
        +1GB
        t
        82
        n
        p
        2
        <ENTER>
        <ENTER>
        a
        2
        w

* Format partitions:

        mkswap /dev/sdb1
        mkfs.ext4 /dev/sdb2

* Mount partitions:

        swapon /dev/sdb1
        mkdir /mnt/exherbo
        mount /dev/sdb2 /mnt/exherbo

* Extract stage3:

        cd /mnt/exherbo
        tar xJpf /mnt/d/install/linux/exherbo-amd64-current.tar.xz

* Mount everything for the chroot:

        mount -o rbind /dev /mnt/exherbo/dev/
        mount -o bind /sys /mnt/exherbo/sys/
        mount -t proc none /mnt/exherbo/proc/

* In case when you to share distfiles across multiple machines (like me), mount distfiles directory:

TODO distfiles.png

        sudo mount -t vboxsf DIST /mnt/exherbo/var/cache/paludis/distfiles -o uid=103,gid=443,umask=007

* Chroot into new system:

        cp /etc/resolv.conf etc/resolv.conf
        env -i TERM=$TERM SHELL=/bin/bash HOME=/root chroot /mnt/exherbo /bin/bash
        source /etc/profile

* Install kernel:

        cd /usr/src
        tar xf /var/cache/paludis/distfiles/linux-3.18.tar.xz
        ln -s linux-3.18 linux
        cd linux
        make menuconfig
        make -j4
        make modules_install
        cp arch/x86_64/boot/bzImage /boot/vmlinuz-3.18
        cp .config /boot/config-3.18
        cp System.map /boot/System.map-3.18
        cd /boot
        ln -s vmlinuz-3.18 vmlinuz
        ln -s config-3.18 config
        ln -s System.map-3.18 System.map

* Set hostname:

        cat <<EOF > /etc/hostname
        exherbo-x64
        EOF

        vi /etc/hosts
        127.0.0.1	localhost	exherbo-x64
        ::1		localhost	exherbo-x64

* Set /etc/machine-id:

        systemd-machine-id-setup

* Comment everything in /etc/vconsole.conf

* Set locale:

        cat <<EOF > /etc/env.d/99locale
        LANG=en_US.utf-8
        EOF

* Update issue file:

        echo "\l (\s \m \r) \t" > /etc/issue

* Install bootloader:

        grub-install /dev/sdb
        cat <<EOF > /boot/grub/grub.cfg
        set timeout=1
        set default=0
        menuentry "Exherbo" {
            insmod ext2
            set root=(hd0,2)
            linux /boot/vmlinuz root=/dev/sda2 rootfstype=ext4 rw quiet
        }
        EOF

* Install root password:

        passwd

* Setup network:

        cat <<EOF > /etc/systemd/network/enp0s3.network
        [Match]
        Name=enp0s3

        [Network]
        DHCP=ipv4
        EOF

* Setup systemd unit for swap partition:

        cat <<EOF > /etc/systemd/network/dev-sda1.swap
        [Swap]
        What=/dev/sda1

        [Install]
        WantedBy=swap.target
        EOF

        systemctl daemon-reload
        systemctl-enable dev-sda1.swap

* Install paludis configuration:

        rm -r /etc/paludis
        git clone https://github.com/medvid/paludis-config.git /etc/paludis

    ** Disable `MULTIBUILD_C: 32`
    ** Resolve circular dependencies in corresponding section in options.conf
    ** Uncomment all lines in multibuild migration section in options.conf
 
 * Sync repositories:
 
        cave sync

* Resolve sys-apps/skeleton-filesystem-layout:

        cave resolve sys-apps/skeleton-filesystem-layout -x1

* Delete /etc/fstab as mount points are handled by systemd

        rm /etc/fstab

* Reinstall glibc (bootstrap option enabled):

        cave resolve sys-libs/glibc -x1

* Reinstall gcc:
 
        cave resolve sys-devel/gcc -x1

* Disable bootstrap option for glibc:

        vi /etc/paludis/options.conf
        cave resolve sys-libs/glibc -x1 --purge sys-devel/bootstrap-gcc

* Enable `MULTIBUILD_C: 32` for all packages:

        vi /etc/paludis/options.conf

* Resolve circular dependencies:

        cave resolve sys-apps/dbus sys-apps/util-linux sys-auth/polkit -x1

* Comment all lines in multibuild migration section in options.conf

* Resolve above packages again

* Resolve world

* Poweroff ArchLinux guest and unmount exherbo-x64.vdi harddisk.

* Start Exherbo guest machine

* Select "Devices" -> "Insert Guest Additions CD image..." from VirtualBox menu

* Install VirtualBox Guest Additions:

        mkdir /mnt/cdrom
        mount /dev/sr0 /mnt/cdrom -o ro
        cd /mnt/cdrom
        #./VBoxLinuxAdditions.run --target /tmp/vbox --noexec
        #cd /tmp/vbox
        ./VBoxLinuxAdditions.run
        cd /opt/VBoxGuestAdditions-*/lib/VBoxGuestAdditions
        cp mount.vboxsf /sbin
        cd /usr/lib/xorg/modules/drivers
        ln -s $OLDPWD/vboxvideo_drv_117.so vboxvideo.so
        ln -s $OLDPWD/vboxmouse_drv_71.so vboxvideo.so
        cd /usr/src/vboxguest-*
        make
        make install

* Install systemd units for VirtualBox:

        cat << EOF > /etc/systemd/system/vbox.service
        [Unit]
        Description=VirtualBox Guest Service

        [Service]
        ExecStartPre=-/sbin/modprobe vboxguest
        ExecStartPre=-/sbin/modprobe vboxvideo
        ExecStartPre=-/sbin/modprobe vboxsf
        ExecStart=/usr/sbin/VBoxService -f

        [Install]
        WantedBy=multi-user.target
        EOF
        
        cat << EOF > /etc/systemd/system/mnt-d.mount
        [Unit]
        Requires=vbox.service
        After=vbox.service

        [Mount]
        What=D_DRIVE
        Where=/mnt/d
        Type=vboxsf
        Options=uid=vmed,gid=users,umask=007

        [Install]
        WantedBy=local-fs.target
        EOF
        
        cat << EOF > /etc/systemd/system/mnt-svn/mount
        [Unit]
        Requires=vbox.service
        After=vbox.service

        [Mount]
        What=SVN
        Where=/mnt/svn
        Type=vboxsf
        Options=uid=vmed,gid=users,umask=007

        [Install]
        WantedBy=local-fs.target
        EOF
        
        cat << EOF > /etc/systemd/system/var-cache-paludis-distfiles.mount
        [Unit]
        Requires=vbox.service
        After=vbox.service

        [Mount]
        What=DIST
        Where=/var/cache/paludis/distfiles
        Type=vboxsf
        Options=uid=paludisbuild,gid=paludisbuild,umask=002

        [Install]
        WantedBy=local-fs.target
        EOF

        cat << EOF > /etc/systemd/system/var-cache-paludis-pbin.mount
        [Unit]
        Requires=vbox.service
        After=vbox.service

        [Mount]
        What=PBIN
        Where=/var/cache/paludis/pbin
        Type=vboxsf
        Options=uid=paludisbuild,gid=paludisbuild,umask=002

        [Install]
        WantedBy=local-fs.target
        EOF

        mkdir -p /mnt/d
        mkdir -p /mnt/svn
        mkdir -p /var/cache/paludis/distfiles
        mkdir -p /var/cache/paludis/pbin

        systemctl daemon-reload
        systemctl enable vbox.service
        systemctl enable mnt-d.mount
        systemctl enable mnt-svn.mount
        systemctl enable var-cache-paludis-distfiles.mount
        systemctl enable var-cache-paludis-pbin.mount

* Add a new user:

        useradd -m -N -G users,systemd-journal,video -s /bin/zsh vmed
        echo "vmed ALL=(ALL) ALL" > /etc/sudoers.d/vmed

* Install systemd unit for xorg autostart:

        cat << EOF > /et/systemd/system/xorg@.service
        [Unit]
        Description=Xorg server on %I
        Documentation=man:Xorg(1)
        After=systemd-user-sessions.service

        Conflicts=getty@%i.service
        After=getty@%i.service

        [Service]
        User=vmed
        WorkingDirectory=/home/vmed
        PAMName=login

        StandardOutput=tty
        StandardInput=tty-fail

        ExecStart=/usr/bin/xinit -- /usr/bin/X vt${XDG_VTNR}
        Restart=always
        RestartSec=0
        TTYPath=/dev/%I
        TTYReset=yes
        TTYVHangup=yes
        TTYVTDisallocate=yes
        IgnoreSIGPIPE=no

        [Install]
        WantedBy=graphical.target
        EOF

        systemctl-daemon-reload
        systemctl enable xorg@tty1.service

* Reboot to user session

* Setup user environment:

        git clone https://github.com/medvid/dotfiles.git ~/dev/projects/dotfiles
        ~/dev/projects/dotfiles/install.sh
        git clone https://github.com/medvid/vimfiles.git ~/.vim
        mv /etc/paludis ~/dev/projects/paludis-config
        sudo chown -Rv vmed:users ~/dev/projects/paludis-config
