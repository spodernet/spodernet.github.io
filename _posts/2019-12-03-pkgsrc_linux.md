---
layout: article
title: NetBSD's pkgsrc as a development environment
tags: linux bsd development
---

As you probabily know I'm in the process of writing my master thesis in CS on _bigraphical reactive systems_. To do
this I'll contribute to, and extend an existing library written in Java, so I needed to setup a development environment.

My goal was to have an isolated environment with its own set of packages and libraries that I could easily manage (add
new packages, remove them and update them) without polluting my "local" environment.
There are various ways to achieve this, for example: 

- by using a __virtual machine__ with the proper packages installed. This solution provides an isolated environment,
  but it does so at the expense of system resources, be it RAM, disk space or CPU. Granted that for my needs it 
  wouldn't have been a big problem since I could have used a minimal Ubuntu or Debian image, but that's still an OS
  worth of disk space on top of the tools and libraries I needed;

- using a containerized environment (__Docker__, __Kubernetes__, ...) provides all benefits of using a virtual machine
  and more, all while taking up less space and being more modular. I already use containers, for example I have one
  container for building $$\LaTeX$$ sources. All in all this is a solid choice;

- __nix__ and __nix-shell__: the nix package manager (and NixOS as a whole) provides a way to define virtual 
  environments using the `nix-shell` command. You can build different environments for different projects and with 
  different version of packages all of them isolated from each other. You can read more about it
  [here](https://nixos.wiki/wiki/Development_environment_with_nix-shell).

Out of these three ways I'd have chosen to use the nix package manager and nix-shell as it is the most flexible of the
them. But that's not what I'm going to talk about, instead I decided to use [pkgsrc](https://www.pkgsrc.org/), NetBSD's
package manager. It offers more than 22.000 packages available for 23 different platforms.

## Setup pkgsrc
Before using pksrc the [wiki](http://wiki.netbsd.org/pkgsrc/how_to_use_pkgsrc_on_linux/) reccomends having the
following packages installed: gcc (and libstdc++), libncurses-devel, zlib and zlib-devel. You can install pkgsrc
using your distribution package manager, for example for Arch Linux you can find it in the
[AUR](https://aur.archlinux.org/packages/netbsd-pkgsrc/), which at the moment of writing this article is based on the
version of packages of Q3 2019. If you want to get the latest version you can use 
[my PKGBUILD](https://git.sr.ht/~spidernet/netbsd-pkgsrc). Otherwise you can install it by downloading it from git, for
example: 
```
git clone --depth 1 -b pkgsrc-2019Q3 https://github.com/netbsd/pkgsrc ~/pkgsrc
```
or `git clone --depth 1 https://github.com/netbsd/pkgsrc ~/pkgsrc` for the latest version.

Once you've installed and downloaded pksrc you have to _bootstrap_ it. It can be bootstrapped in two modes: privileged
and unprivileged. In unprivileged mode packages are installed for one single user. By default it'd be bootstrapped in
privileged moded. Boostraping it is as simple as:
```
$ cd /path/to/pkgsrc/bootstrap
# ./bootstrap
```
To bootrstrap it in unprivileged mode you can pass it the `--unprivileged` option. By default, in privileged mode pksrc
will use `/usr/pkg` for _prefix_ where programs will be installed, while in unprivileged mode `~/pkg` will be used. To
manually change the prefix you can use the `--prefix` option. It is thus possibile to bootstrap multiple instances of
pkgsrc in different directories and have different development environments.

After having bootstrapped pkgsrc we can set the `PATH` and `MANPATH` accordingly:
```
$ PATH=/usr/pkg/sbin:/usr/pkg/bin:$PATH
$ MANPATH=/usr/pkg/man:$MANPATH
```

If we have multiple instances of pkgsrc we have to be extra careful about setting the `PATH` variable. For example if
we have two instances of pkgsrc, one for Java development and one for C development in folders `~/javaenv` and
`~/cenv`, by using some bash magic we can define two functions to set the `PATH` variable whenever we want to use the 
binaries in  the Java environment or in the C environment, or even change the path depending on the current working 
directory (see [here](https://superuser.com/questions/914087/can-i-change-my-path-dynamically-based-on-my-cwd)).

## Manage packages
Pkgsrc provides a series of utilities to help you manage your packages:

- `pkg_info` lets you query information about installed packages. For example `pkg_info -u` lists all user-installed
  packages and not their dependencies, while `pkg_info -a` shows all isntalled packages, for more info read `man
  pkg_info`;

- `pkg_admin` lets you perform various administration tasks on your packages, like check the MD5 checksum of installed
  packages, check vulnerabilities of installe packages and other tasks.

The other utilities will be discussed in the following sections.

### Install packages
There are two ways to install packages: you can either build it from source or install a precompiled binary.

#### Build packages
To build a package from sources you first need to locate it under `${PREFIX}/pkgsrc`, for example to search `clang` you
could do
```bash
$ find /usr/pkgsrc -type d -name "clang"
/usr/pkgsrc/lang/clang
```
Now to build it you would `cd` into `/usr/pkgsrc/lang/clang` and then you can compile and install it:
```bash
$ sudo bmake install
```
this would also build its dependencies. If you want to clean the object files generates during the build process you
could append `clean` to the build command: `sudo bmake install clean`.

If you're building a package for several systems you can create a package and install it afterwards:
```bash
$ sudo bmake package
[...]
=> Creating binary package in /usr/pkgsrc/packages/All/clang-X.Y.Z.tgz

$ sudo pkg_add /usr/pkgsrc/packages/All/clang-X.Y.Z.tgz
```

#### Install binaries
In order to install precompiled binaries we need to use [pkgin](http://pkgin.net/), we can install it by building it
from source by calling `sudo bmake install clean` from `${PREFIX}/usr/pkgsrc/pkgtools/pkgin`.

Before using pkgin we need to setup a repository in the `${PREFIX}/etc/pkgin/repositories.conf` file, we will use
the binary packages provided by [joyent](https://pkgsrc.joyent.com/):
```bash
$ echo https://pkgsrc.joyent.com/packages/Linux/el7/trunk/x86_64/All/ > /usr/pkg/etc/pkgin/repositories.conf
```
After this you can use `# pkgin update` to build the initial database and to keep it synchronized. Now you're ready to
use pkgin, for example to install tmux run: `# pkgin install tmux`. To learn about the other commands you can refer its
manual or to [this page](http://pkgin.net/).

### Remove packages
Removing a package, whether it was installed from source code or from a binary package is as simple as calling 
```bash
$ sudo pkg_delete pkg_name
```
it can also accept wildcards, for example `# pkg_delete "*emacs*"` removes the set of emacs packages.

### Update packages
There are [various ways](https://wiki.netbsd.org/pkgsrc/how_to_upgrade_packages/) to update packages in pkgsrc. For
updating binaries I use:
```bash
$ pkgin update
$ pkgin full-upgrade
```
For packages built from source I use `pkg_rolling-replace` a shell script available in 
[`pkgtools/pkg_rolling-replace`](http://pkgsrc.se/pkgtools/pkg_rolling-replace). For more info check its man page or 
[this page](https://wiki.netbsd.org/pkgsrc/how_to_upgrade_packages/) in pkgsrc's wiki.

__N.B.:__ before updating packages make sure to download an updated `pkgsrc` directory.

## Pros and Cons
We've seen how to setup pkgsrc, how to install packages and how it could be used as a development environment and with
some effort we can have multiple environments. But there are some disadvantages to consider: first of all, not
every package is available as a binary (for Linux) so we might have to compile it from source, second is that as far as I
am aware you cannot directly install a specific version of a package that is not in the repository. Moreover, as we've 
seen, setting up multiple development environments is a bit involved and requires to mess with environment variables, 
so it's not really user friendly.
All these disadvantages come from the fact that pkgsrc's goal is to be a package management system and not to provide a
development environment.

## Conclusion
In conclusion it's indeed possible to use pkgsrc as a development environments but it is a quite exotic setup and it
has some shortcomings, therefore it might not be for everyone or even suitable especially in production.
