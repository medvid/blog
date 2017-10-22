---
title: Patching interpreter for third-party elf binaries
date: 2015-11-08 00:00:00 Z
---

    cave resolve dev-util/patchelf::heirecka
    ELF_INTERPRETER=/usr/x86_64-pc-linux-gnu/lib/ld-linux-x86-64.so.2
    patchelf --set-interpreter ${ELF_INTERPRETER} /usr/x86_64-pc-linux-gnu/bin/VBoxService
    patchelf --set-interpreter ${ELF_INTERPRETER} /usr/x86_64-pc-linux-gnu/bin/VBoxClient
    patchelf --set-interpreter ${ELF_INTERPRETER} /usr/x86_64-pc-linux-gnu/bin/VBoxControl
    patchelf --set-interpreter ${ELF_INTERPRETER} /usr/x86_64-pc-linux-gnu/bin/vbox-greeter
