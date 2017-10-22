---
title: Setting up pbin repository for paludis
date: 2015-01-31 00:00:00 Z
aliases:
- "/exherbo/2015/01/31/pbin/"
---

## Overview

This note contains instructions how to setup binary package repository for the
[Paludis Package Mangler][paludis].

## Disclaimer

This guide is unofficial. It may be incorrect, out of date, or harmful in other ways.
Please see [Ciaran's mail][ciaran] for the reasoning.

If you don't want to make Ciaran sad, please close this page and
refer to [the official pbin documentation][pbins].
I hope you will find it useful (heh!).

## Rationale

Binary packages (pbins) may be useful in case when you have multiple exherbo
setups (say, chroots) and you want to share pre-built packages between them.
To benefit from this approach, you should have the same environment on
different hosts (architecture, compilation flags, build options, etc).
Sharing binary packages built with `CFLAGS="march=native"` across machines
with different CPUs is definitely a bad idea.
Personally, I use pbins to populate exherbo containers with systemd-nspawn,
so I'm safe here.

## Prerequisites

paludis should be built with pbin support ([exherbo default][default]):

    sys-apps/paludis pbin

## Setting up pbin repository

You should create all required repository files manually:

  * Go to directory where repositories are stored:

        cd /var/db/paludis/repositories

  * Create directory for binary package repository (name it just 'pbin'):

        mkdir pbin

  * Set appropriate permissions:

        chown paludisbuild:paludisbuild pbin
        chmod g+w pbin

  * Prepare repository tree layout (see [tree layout guide][efs]):

        cd pbin
        mkdir metadata packages profiles
        echo 'pbin' > profiles/repo_name
        touch profiles/options.conf
        touch metadata/categories.conf

When done, store repository configuration at /etc/paludis/repositories/pbin.conf:

    format = e
    location = /var/db/paludis/repositories/pbin
    profiles = ${location}/profiles
    distdir = /var/cache/paludis/pbin
    binary_destination = true
    binary_distdir = ${distdir}
    binary_keywords_filter = amd64 ~amd64
    tool_prefix = x86_64-pc-linux-gnu-

Option `distdir` specifies a directory where actual binary packages (`*.tar.bz2`) are stored.
`location` specifies a path to pbin repository with exhereses for these packages.
These exhereses are automatically updated by paludis when you create binary packages.
When you're done with initial setup, you may forget about this repository.

## Checking that everything works

To make sure that pbin repository is set up correctly,
create some binary package and install it with `--via-binary` cave option:

    cave resolve --via-binary '*/*' sydbox
    Done: 1676 steps

    These are the actions I will take, in order:

    n   sys-apps/sydbox:0::arbor scm to ::pbin
        "Sydbox the other sandbox"
        doc seccomp build_options: symbols=split jobs=2 -dwarf_compress recommended_tests -trace work=tidyup
        Reasons: target (to be like sys-apps/sydbox:0::(install_to_slash))

    r   sys-apps/sydbox:0::arbor scm to ::installed via binary created in pbin replacing scm
        doc seccomp build_options: symbols=split jobs=2 -dwarf_compress recommended_tests -trace work=tidyup
        Reasons: target

    Total: 1 reinstalls, 1 binaries

As you can see, paludis automatically creates pbin package and selects it for install.

When you try to install the same package again,
you'll see that there is no need to rebuild anything again:

    cave resolve sydbox
    Done: 1670 steps

    These are the actions I will take, in order:

    r   sys-apps/sydbox:0::pbin scm to ::installed replacing scm
        (doc) (seccomp) build_options: (-recommended_tests) -trace work=tidyup
        Reasons: target

    Total: 1 reinstalls


## Settings repository priorities

Sometimes package rebuild is required. In this case you'll need to specify
package repository manually:

    cave resolve sydbox::arbor

If you don't want to specify original repository every time when rebuild is required,
you can lower importance of pbin repository:

    echo "importance = -1" >> /etc/paludis/repositories/pbin.conf

Default repository importance is 0, so original repository always takes precedence
over when pbin unless `--via-binary` option is used:

    cave resolve sydbox
    Done: 1670 steps

    These are the actions I will take, in order:

    r   sys-apps/sydbox:0::arbor scm to ::installed replacing scm
        doc seccomp build_options: symbols=split jobs=2 -dwarf_compress recommended_tests -trace work=tidyup
        Reasons: target

    Total: 1 reinstalls

Compare with:

    cave resolve sydbox --via-binary '*/*'
    Done: 1677 steps

    These are the actions I will take, in order:

    r   sys-apps/sydbox:0::arbor scm to ::pbin replacing scm
        doc seccomp build_options: symbols=split jobs=2 -dwarf_compress recommended_tests -trace work=tidyup
        Reasons: target (to be like sys-apps/sydbox:0::(install_to_slash))

    r   sys-apps/sydbox:0::arbor scm to ::installed via binary created in pbin replacing scm
        doc seccomp build_options: symbols=split jobs=2 -dwarf_compress recommended_tests -trace work=tidyup
        Reasons: target

You can specify pbin repository in an explicit way to skip rebuild:

    cave resolve sydbox::pbin
    Done: 1670 steps

    These are the actions I will take, in order:

    r   sys-apps/sydbox:0::pbin scm to ::installed replacing scm
        (doc) (seccomp) build_options: (-recommended_tests) -trace work=tidyup
        Reasons: target

    Total: 1 reinstalls

Option `--promote-binaries if-same` does the same:

    cave resolve sydbox --promote-binaries if-same
    Done: 1670 steps

    These are the actions I will take, in order:

    r   sys-apps/sydbox:0::pbin scm to ::installed replacing scm
        (doc) (seccomp) build_options: (-recommended_tests) -trace work=tidyup
        Reasons: target

    Total: 1 reinstalls

Last command brings additional convenience: it uses pbin package when present,
and builds from source otherwise.

## Making binary packages without installation

To add binary package to pbin repository without installing it, run cave with `-mb`
argument:

    cave resolve sydbox -mb

If you want to create binary versions of all installed packages, run:

    cave resolve world -c -mb

It is impossible to use `installed-packages` built-in set because it also contains
installed accounts items (creation of pbins for user or group packages is impossible).

## Final steps

For convenience, you can setup shell aliases to build pbins and install
binary packages automatically:

    # install package from source
    alias crs="sudo cave resolve"
    # create pbin and install it
    alias crb="crs --via-binary '*/*'"
    # use pbin when already built
    # this is the default alias to be used for installing packages
    alias cr="crb --promote-binaries if-same"

These and others cave aliases can be found at [my zsh configuration repo][prezto].

## Feedback

If you have any feedback, please email me and I will update the article.


[paludis]: http://paludis.exherbo.org
[pbins]: http://paludis.exherbo.org/overview/pbins.html
[ciaran]: http://lists.exherbo.org/pipermail/exherbo-dev/2010-March/000683.html
[default]: http://lists.exherbo.org/pipermail/exherbo-dev/2014-April/001310.html
[efs]: http://www.exherbo.org/docs/exheres-for-smarties.html#tree_layout
[prezto]: https://raw.githubusercontent.com/medvid/prezto/master/modules/exherbo/init.zsh
