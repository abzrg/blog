---
title: "Installing OpenFOAM (ESI v2206) on an M1 Mac"
date: 2022-09-23T14:04:20+03:30
draft: false
tags: ['OpenFOAM', 'swak4Foam']
# categories: ['programming']
---

<!-- ---------------------------------------------------------------------- -->

## Case-sensitivity

Nowadays, Apple Macbooks don’t ship with a case-sensitive file system.
However, the OpenFOAM source code heavily uses case-sensitive names for conveying different meanings.
We first need to create acase-sensitive partition/volume.
For that, we use the Disk Utility program, per below.

![The selected settings for Disk Utility](/img/openfoam_installation_disk_utility.png)

Provide a name (This will be your `FOAM_INST_DIR` directory which contains all OpenFOAM installations).
Here I specified it as `OpenFOAM`.
For the format, we need to choose a **case-sensitive** format (In this case, `APFS (case-sensitive)`).

Now, we have a partition on our disk with a case-sensitive file system.
It is mounted on `/Volumes/OpenFOAM`, but navigating there is a bit inconvenient.
To have it in the `HOME` directory, we create a symbolic link.

```
ln -s /Volumes/OpenFOAM $HOME/OpenFOAM
```

<!-- ---------------------------------------------------------------------- -->

## Installing Prerequisites using Homebrew

```
brew install git cmake open-mpi libomp bear paraview
```
> - `git`: to clone (download) source code and applying patch (for swak4Foam)
> - `cmake`: to make build files to be consumed by make
> - `openmpi`, `libomp`: Runtime parallel libraries (for parallel runs and ...)
> - `bear`: Generate compilation database for clang tooling (clangd)
> - `paraview`: Data analysis and visualization application

<!-- ---------------------------------------------------------------------- -->

## Get the Source Code

We can grab the source code by cloning the git repository from GitLab

```
git clone https://develop.openfoam.com/Development/openfoam
```

This clone is pointing to the development branch.
However, here I’ll choose the latest "stable" version, the `OpenFOAM-v2206`. 

```
mv openfoam OpenFOAM-v2206
cd $_
git checkout OpenFOAM-v2206
```

<!-- ---------------------------------------------------------------------- -->

## Shell config (Optional)

Now that we have the desired version,
we can tweak a bunch of settings and files before proceeding to build OpenFOAM.
Note that we should not edit config files under the version control
because those changes will be discarded upon updates (for example, using `git pull`).
OpenFOAM
[suggests](https://develop.openfoam.com/Development/openfoam/-/wikis/configuring)
putting a user-defined configuration in the `$HOME/.OpenFOAM` directory.
We also create the `2206` in that directory to make it version-specific.

```
mkdir -p $HOME/.OpenFOAM/2206
```

Next, put your desired config there. Here's mine

```
# Setting the size of `label` to 64 bit
export WM_LABEL_SIZE=64

# If you want to debug your code
export WM_COMPILE_OPTION=Debug

# Some usefull aliases
alias foamfv='cd $FOAM_SRC/finiteVolume'
alias foamsrc='cd $FOAM_SRC/OpenFOAM'
alias foam3rdParty='cd $WM_THIRD_PARTY_DIR'
alias paraFOAM='paraFOAM -builtin'
```

Also, we need to get rid of one compiler warning,
`-Wunsupported-floating-point-opt`. For that we edit the file
`$WM_PROJECT_DIR/wmake/rules/general/Clang/c++` and add the
`Wno-unsupported-floating-point-opt` to the end of `c++WARN`
variable. (Notice the `\` at the line after the
`-Wno-unknown-warning-option`)

```
# ~~~
c++WARN     = \
    -Wall -Wextra -Wold-style-cast \
    -Wnon-virtual-dtor -Wno-unused-parameter -Wno-invalid-offsetof \
    -Wno-undefined-var-template \
    -Wno-unknown-warning-option \
    -Wno-unsupported-floating-point-opt
# ~~~
```

> I couldn't find a way to add this flag in a file in the `$HOME/.OpenFOAM` directory.

<!-- ---------------------------------------------------------------------- -->

## Build

First, we navigate to the `OpenFOAM-v2206` and load the OpenFOAM environment

```
cd ~/OpenFOAM/OpenFOAM-v2206/
source etc/bashrc
```

Then we run the `foamSystemCheck` shell script to check the
machine, software components, and the environment for installing
OpenFOAM

```
foamSystemCheck
```

> Hopefully, it won’t create any errors because I didn’t face any! ;).

Next, we run the installation script, `Allwmake`, with a few flags

```
./Allwmake -j -s -l -with-bear
```
> - You can check what is the meaning of these flags with `wmake -help`command
> - `-with-bear` is optional unless you are using an `LSP`
> - `-j` will utilize all your CPU cores. To only use a certain number of cores: `-j 2`


If the build has finished without any fatal errors, installation can
be tested with the following command

```
foamInstallationTest
```

<!-- ---------------------------------------------------------------------- -->

## swak4Foam

To install the swak4Foam package on macOS, first, we need to patch it.

```
curl https://gist.githubusercontent.com/reverseila/39101060b5f05d9b2d2d151a39fda158/raw/eca1be96c534ae0eb4938c5491028d8d7838328b/swak4Foam.patch | git apply
```

Then, we need to install three more prerequisites. `bison` on macOS is
outdated for building swak4Foam. So we install a new version of
`bison` with homebrew and add it to `PATH`

```
brew install bison pkgconfig
export PATH="$(brew --prefix)/opt/bison/bin:$PATH"
```
> - `bison`: Parser generator
> - `pkgconfig`: Manage compile and link flags for libraries

Further, The `xargs` utility used inside the scripts of the
`swak4FOAM` uses a specific feature of the GNU version of `xargs` that
produces an error with macOS version of `xargs`.
We need to install the `findutils` package and add it to `PATH`.

```
brew install findutils

export PATH="/opt/homebrew/opt/findutils/libexec/gnubin:$PATH"
```

Next, we build it

```
./AllwmakeAll 2>&1 | tee log.AllwmakeAll
```

We can test the installation by running a swak4Foam utility. For example

```
funkySetFields -help
```

<!-- ---------------------------------------------------------------------- -->

## Resources:

1. [OpenFOAM ESI group](https://develop.openfoam.com/Development/openfoam)
1. [BrushXue/OpenFOAM-AppleM1](https://github.com/BrushXue/OpenFOAM-AppleM1)
1. [OpenFOAM Configuration System](https://develop.openfoam.com/Development/openfoam/-/wikis/configuring)
