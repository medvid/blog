---
title: How to see which flags -march=native will activate
date: 2015-02-01
---

* [stackoverflow][so]

* View flags

    gcc -march=native -E -v - </dev/null 2>&1 | grep cc1

* View descriptions

    gcc -march=native -Q --help=target ...gcc -march=native -Q --help=target ...

* View macro definitions

    echo | gcc -dM -E - -march=native

* View processor features

    cat /proc/cpuinfo

* Native flags for [Intel(R) Pentium(R) Dual  CPU  E2200  @ 2.20GHz][crap]

    * gcc

        /usr/libexec/gcc/x86_64-pc-linux-gnu/4.9.2/cc1 -E -quiet -v - -march=core2 -mmmx -mno-3dnow -msse -msse2 -msse3 -mssse3 -mno-sse4a -mcx16 -msahf -mno-movbe -mno-aes -mno-sha -mno-pclmul -mno-popcnt -mno-abm -mno-lwp -mno-fma -mno-fma4 -mno-xop -mno-bmi -mno-bmi2 -mno-tbm -mno-avx -mno-avx2 -mno-sse4.2 -mno-sse4.1 -mno-lzcnt -mno-rtm -mno-hle -mno-rdrnd -mno-f16c -mno-fsgsbase -mno-rdseed -mno-prfchw -mno-adx -mfxsr -mno-xsave -mno-xsaveopt -mno-avx512f -mno-avx512er -mno-avx512cd -mno-avx512pf -mno-prefetchwt1 --param l1-cache-size=32 --param l1-cache-line-size=64 --param l2-cache-size=1024 -mtune=core2

    * clang

        "/usr/bin/clang-3.5" -cc1 -triple x86_64-pc-linux-gnu -E -disable-free -main-file-name - -mrelocation-model static -mdisable-fp-elim -fmath-errno -masm-verbose -mconstructor-aliases -munwind-tables -fuse-init-array -target-cpu core2 -v -dwarf-column-info -resource-dir /usr/bin/../lib/clang/3.5.1 -internal-isystem /usr/local/include -internal-isystem /usr/bin/../lib/clang/3.5.1/include -internal-externc-isystem /include -internal-externc-isystem /usr/include -fdebug-compilation-dir /home/vmed/dev/projects/medvid.github.io -ferror-limit 19 -fmessage-length 0 -mstackrealign -fobjc-runtime=gcc -fdiagnostics-show-option -o - -x c -


[so]: http://stackoverflow.com/questions/5470257/how-to-see-which-flags-march-native-will-activate
[crap]: http://ark.intel.com/products/33925/Intel-Pentium-Processor-E2200-1M-Cache-2_20-GHz-800-MHz-FSB
