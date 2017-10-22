---
title: Bootstrapping exherbo on Pinebook
date: 2017-07-09 00:00:00 Z
aliases:
- "/exherbo/2017/07/09/aarch64/"
---

Based on https://exherbo.org/docs/bootstrapping.html

    sudo apt install m4 autoconf automake xmlto tidy cmake libtool libjansson-dev file libmagic-dev libpcre3-dev texinfo liblz-dev zlib1g-dev libgmp-dev libmpfr-dev libmpc-dev
    export CHOST=aarch64-unknown-linux-gnueabi
    export PATH=/usr/${CHOST}/bin:${PATH}
    mkdir /mnt/exherbo/
    
    git clone https://git.exherbo.org/eclectic.git
    cd eclectic
    ./autogen.bash
    ./configure --build=${CHOST} --host=${CHOST}          \
              --prefix=/usr/${CHOST}                    \
              --bindir=/usr/${CHOST}/bin                \
              --sbindir=/usr/${CHOST}/bin               \
              --libdir=/usr/${CHOST}/lib                \
              --datadir=/usr/share                      \
              --datarootdir=/usr/share                  \
              --docdir=/usr/share/doc/eclectic-scm      \
              --infodir=/usr/share/info                 \
              --mandir=/usr/share/man                   \
              --sysconfdir=/etc                         \
              --localstatedir=/var/lib
    make
    make install
    cd ..
    
    git clone git://git.exherbo.org/paludis/paludis.git
    cd paludis
    mkdir build
    cd build
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr/${CHOST} -DENABLE_GTEST:BOOL=FALSE -DENABLE_DOXYGEN:BOOL=FALSE -DENABLE_PBINS:BOOL=FALSE -DPALUDIS_COLOUR_PINK:BOOL=FALSE -DENABLE_PYTHON:BOOL=FALSE -DENABLE_PYTHON_DOCS:BOOL=FALSE -DENABLE_RUBY:BOOL=FALSE -DENABLE_RUBY_DOCS:BOOL=FALSE -DENABLE_SEARCH_INDEX:BOOL=FALSE -DENABLE_VIM:BOOL=FALSE -DENABLE_XML:BOOL=FALSE -DCMAKE_INSTALL_SYSCONFDIR=/etc -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/paludis-scm -DCMAKE_INSTALL_DATAROOTDIR:PATH=/usr/share/ -DPALUDIS_VIM_INSTALL_DIR=/usr/share/vim/vimfiles -DPALUDIS_CLIENTS="cave" -DPALUDIS_ENVIRONMENTS="default;test" -DPALUDIS_REPOSITORIES="all" -DPALUDIS_DEFAULT_DISTRIBUTION=exherbo -DCONFIG_FRAMEWORK=eclectic -DENABLE_STRIPPER:BOOL=FALSE -DBUILD_SHARED_LIBS:BOOL=TRUE -DUSE_PREBUILT_DOCUMENTATION:BOOL=FALSE
    make
    make install
    cd ..
    cd ..
    
    usermod -u 301 systemd-bus-proxy
    groupadd -g 443 paludisbuild
    useradd -d /var/tmp/paludis -G tty -g paludisbuild -u 103 paludisbuild
    
    cd eclectic
    ./autogen.bash
    ./configure --build=${CHOST} --host=${CHOST}          \
              --prefix=/usr/${CHOST}                    \
              --bindir=/usr/${CHOST}/bin                \
              --sbindir=/usr/${CHOST}/bin               \
              --libdir=/usr/${CHOST}/lib                \
              --datadir=/usr/share                      \
              --datarootdir=/usr/share                  \
              --docdir=/usr/share/doc/eclectic-scm      \
              --infodir=/usr/share/info                 \
              --mandir=/usr/share/man                   \
              --sysconfdir=/etc                         \
              --localstatedir=/var/lib
    make
    make install
    cd ..
    
    add to /etc/paludis/bashrc
    FILESYSTEM_LAYOUT="cross"
    CHOST="aarch64-unknown-linux-gnueabi"
    aarch64_unknown_linux_gnueabi_CFLAGS="-pipe -O2 -g -march=armv8-a+crc -mtune=cortex-a53"
    aarch64_unknown_linux_gnueabi_CXXFLAGS="-pipe -O2 -g -march=armv8-a+crc -mtune=cortex-a53"
    export PATH="/usr/${CHOST}/bin:${PATH}"

    add to /etc/paludis/general.conf
    world = ${root}/etc/paludis/world

    add to /etc/paludis/licences.conf
    */* *

    add to /etc/paludis/options.conf
    */* targets: aarch64-unknown-linux-gnueabi
    */* build_options: jobs=2 -recommended_tests symbols=preserve
    */* providers: links -openssl libressl -pkg-config pkgconf
    */* -python -ruby -perl

    add to /etc/paludis/platforms.conf
    */* aarch64 ~aarch64 *

    add to /etc/paludis/repository.template
    format = %{repository_template_format}
    location = /var/db/paludis/repositories/%{repository_template_name}
    sync = %{repository_template_sync}

    mkdir /etc/paludis/repositories
    cd repositories

    add to accounts.conf
    format = accounts

    add to arbor.conf
    location = ${root}/var/db/paludis/repositories/arbor
    sync = git+https://git.exherbo.org/arbor.git
    profiles = ${location}/profiles
    format = e
    names_cache = ${root}/var/cache/paludis/names
    write_cache = ${root}/var/cache/paludis/metadata

    add to installed.conf
    location = ${root}/var/db/paludis/repositories/installed
    format = exndbam
    names_cache = ${root}/var/cache/paludis/names
    split_debug_location = /usr/aarch64-unknown-linux-gnueabi/lib/debug
    tool_prefix = aarch64-unknown-linux-gnueabi-

    add to installed_accounts.conf
    format = installed-accounts
    handler = passwd

    add to repository.conf
    format = repository
    config_filename = /etc/paludis/repositories/%{repository_template_name}.conf
    config_template = /etc/paludis/repository.template

    mkdir /var/tmp/exherbo-tools
    cd /var/tmp/exherbo-tools
    ln -s /usr/bin/gcc-ar aarch64-unknown-linux-gnueabi-ar
    ln -s /usr/bin/as aarch64-unknown-linux-gnueabi-as
    ln -s /usr/bin/c++ aarch64-unknown-linux-gnueabi-c++
    ln -s /usr/bin/cc aarch64-unknown-linux-gnueabi-cc
    ln -s /usr/bin/cpp aarch64-unknown-linux-gnueabi-cpp
    ln -s /usr/bin/g++ aarch64-unknown-linux-gnueabi-g++
    ln -s /usr/bin/gcc aarch64-unknown-linux-gnueabi-gcc
    ln -s /usr/bin/ld aarch64-unknown-linux-gnueabi-ld
    ln -s /usr/bin/nm aarch64-unknown-linux-gnueabi-nm
    ln -s /usr/bin/objcopy aarch64-unknown-linux-gnueabi-objcopy
    ln -s /usr/bin/objdump aarch64-unknown-linux-gnueabi-objdump
    ln -s /usr/bin/pkg-config aarch64-unknown-linux-gnueabi-pkg-config
    ln -s /usr/bin/ranlib aarch64-unknown-linux-gnueabi-ranlib
    ln -s /usr/bin/readelf aarch64-unknown-linux-gnueabi-readelf

    edit /etc/paludis/bashrc
    export PATH="/usr/${CHOST}/bin:/var/tmp/exherbo-tools:${PATH}"

    add to /etc/env.d/00basic
    PATH="/opt/bin"
    LDPATH="/usr/local/lib"
    MANPATH="/usr/local/share/man:/usr/share/man"
    INFOPATH="/usr/share/info"
    CVS_RSH="ssh"
    PAGER="/usr/host/bin/less"

    add to /etc/paludis/package_mask.conf
    >=sys-kernel/linux-headers-3.11
    >=sys-apps/iproute2-3.11


    add to ~/.bashrc and /etc/paludis/bashrc
    export LD_LIBRARY_PATH=/usr/aarch64-unknown-linux-gnueabi/lib

    git clone git://git.exherbo.org/arbor.git /var/db/paludis/repositories/arbor
    mkdir /var/db/paludis/repositories/installed
    mkdir /var/tmp/paludis/build -p
    chown paludisbuild:paludisbuild /var/tmp/paludis/build
    chmod g+w /var/tmp/paludis/build

    mkdir -p /var/cache/paludis/names
    mkdir /var/cache/paludis/distfiles
    chown 103:443 /var/cache/paludis/distfiles
    chmod g+w /var/cache/paludis/distfiles
    cave fix-cache
    mkdir /var/cache/paludis/metadata
    add to ~/.bashrc and /etc/paludis/bashrc
    export PALUDIS_DO_NOTHING_SANDBOXY=1
    cave generate-metadata
    mkdir /var/lib/exherbo/news -p
    touch /var/lib/exherbo/news/news-arbor.skip
    mkdir /var/log/paludis

    echo alias cr="cave resolve -z1 -0 '*/*' -x1" >> ~/.bashrc
    source ~/.bashrc
    cr binutils
    cr mpfr
    cr gmp
    cr mpc
    cr linux-headers
    cr glibc

FUCK THIS SHIT, it is not going to work. Merging glibc turns the host system into a bloody mess.
With the default dynamic linker, cave segfaults somewhere deep in the libstdc++ jungles.
With the just bootstrapped linker, any dynamic executable from /usr/aarch64-unknown-linux-gnueabi/bin refuses to start, with an undecipherable error message.

FUCK IT.
