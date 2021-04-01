+++
title = "Install"
toc = false
type = "docs"
weight = 10

[menu.doc]
weight = 1
parent = "setup"
+++

This page describes how to set up and build the project on Ubuntu (20.04 LTS) and Arch Linux.

# Prerequisites

First, dependencies to build the project need to be installed.

## Ubuntu (20.04 LTS)

The packages given here were tested under Ubuntu 20.04 LTS, but could also work on later versions of Ubuntu, possibly requiring adjustments.

Install the following required packages:

```console
# apt install git pipenv cmake ccache rsync ninja-build libgmp-dev autoconf
```

### Python 3.9

Building the project requires Python 3.9, which is (at the time of writing) not available in the official Ubuntu 20.04 package repositories.
Since multiple ways to acquire this Python version exist (e.g., using a user-provided ppa, installing from source or using a tool such as Pyenv), we leave this part to the user, and assume Python 3.9 to be installed in the rest of this guide.

### Rustup

[Rustup](https://rustup.rs/), a toolchain manager for the Rust programming language, is also required to build the project, but not available in the Ubuntu 20.04 repositories.

To install, follow the instructions given on [https://rustup.rs/](https://rustup.rs/), which, at the time of writing, are (might also require installing `curl`):

```console
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

And follow the installer that launches.

Rustup requires additional configuration to build the project, follow the [instructions below](#rustup-configuration).

## Arch Linux

The following packages are required to build the project:

```console
# pacman -S git openssh python python-pipenv base-devel cmake ninja ccache rsync rustup
```

Rustup requires additional configuration, follow the [instructions below](#rustup-configuration).

## Rustup configuration

On both Ubuntu and Arch Linux rustup requires additional configuration to build the project.

To do so, run the following once rustup is installed (which should be done after the OS-specific steps above are completed):

```console
$ rustup install nightly
$ rustup component add --toolchain nightly rust-src
```

# Building the project

Once all dependencies are installed, we can fetch and build the project.
We assume that you have access to a git repository from which you can pull the workspace.

```console
$ git clone <git repo link> workspace
$ cd workspace
$ ./ws build
```

Running `./ws build` can take some time when first run, as it will fetch all sources and build them, which includes, e.g., building a version of LLVM.
Subsequent usages of `./ws build` will run incrementally, only recompiling what needs to be recompiled dependening on changes that were made (if any).
Also see the `README.md` file in the repository, it gives additional usage instructions for the `./ws` command.