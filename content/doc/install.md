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

The packages given here were tested under Ubuntu 20.04 LTS, but probably also work on later versions of Ubuntu, maybe requiring slight adjustments.

Install the following required packages:

```bash
# apt install git pipenv cmake ccache rsync ninja-build libgmp-dev autoconf
```

### Python 3.9

Building the project requires Python 3.9, which is (at the time of writing) not available in the official Ubuntu 20.04 package repositories.
There are multiple ways to install this version of Python on Ubuntu, one possible way would be to use a ppa that provides this version:

```bash
# apt install software-properties-common
# add-apt-repository ppa:deadsnakes/ppa
# apt install python3.9
```

Another way would be to, e.g., use [Pyenv](https://medium.com/@marine.ss/installing-pyenv-on-ubuntu-20-04-c3a609a20aa2) to install Python 3.9.

### Rustup

[Rustup](https://rustup.rs/), a toolchain manager for the Rust programming language, is also required to build the project, but not available in the Ubuntu 20.04 repositories.

To install, follow the instructions given on [https://rustup.rs/](https://rustup.rs/), which, at the time of writing, are:

```bash
$ apt install curl
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

And follow the installer that launches.

Rustup requires additional configuration to build the project, which is described further down in this page.

## Arch Linux

The following packages are required to build the project:

```bash
# pacman -S git openssh python python-pipenv base-devel cmake ninja ccache rsync rustup
```

Additionally, rustup requires additional configuration, which is described below.

## Rustup configuration

On both Ubuntu and Arch Linux rustup requires additional configuration to build the project.

To do so, run the following once rustup is installed (which should be done after the OS-specific instructions above are completed):

```bash
$ rustup install nightly
$ rustup component add --toolchain nightly rust-src
```

# Building the project

Once all dependencies are installed, we can fetch and build the project.
We assume that you have access to a git repository from which you can pull the workspace.

```bash
$ git clone <git repo link> workspace
$ cd workspace
$ ./ws build
```

Running `./ws build` can take some time when first run, as it will fetch all sources and build them, which includes, e.g., building a version of LLVM.
Subsequent usages of `./ws build` will run incrementally, only recompiling what needs to be recompiled dependening on changes that were made (if any).
Also see the `README.md` file in the repository, it gives additional usage instructions for the `./ws` command.