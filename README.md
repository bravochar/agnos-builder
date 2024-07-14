# agnos-builder

This is the tool to build AGNOS, our Ubuntu based OS. AGNOS runs on the [comma three devkit](https://comma.ai/shop/products/three).

NOTE: the `edk2_tici` and `agnos-firmware` submodules are internal, private repos.

## Setup

These tools are developed on and targeted for Ubuntu 20.04.

Run once to set things up:
```sh
git submodule update --init agnos-kernel-sdm845
./tools/extract_tools.sh
```

### MacOS (M1)

Install Git LFS (if needed)

```sh
brew install git-lfs
```

`pyenv` can be used to provide `python2`:

```sh
brew install pyenv
pyenv install 2.7.18
pyenv install 3.12.4	# also needed
pyenv init

# follow instructions to append to `.zshrc` and restart your shell

# sets global versions for `python2` and `python3` commands
pyenv global 3.12.4 2.7.18
```

Continue as above:

```sh
git submodule update --init agnos-kernel-sdm845
```

#### M1 version of all build tools :(((

We will need Mac versions of all of the build tools - because M1 is ARM-like and not x86_64

 * [ ] LINARO_GCC=aarch64-linux-gnu-gcc
 * [ ] GOOGLE_GCC_4_9=aarch64-linux-android-4.9
 * [ ] EDK2_LLVM=llvm-arm-toolchain-ship
 * [ ] SEC_IMAGE=SecImage

TODO: get these upstreamed in LFS

#### Download latest bare-metal gcc from linaro for m1 MAC

Download tarball [here](https://developer.arm.com/-/media/Files/downloads/gnu/13.3.rel1/binrel/arm-gnu-toolchain-13.3.rel1-darwin-arm64-aarch64-none-elf.tar.xz), and extract contents into `/tools/aarch64-darwin-none-elf-gcc`. `build_kernel.sh` will look for it there.

Install and link openssl headers/libraries

```sh
brew install openssl

# symlink stuff
sudo mkdir /usr/local/include
sudo mkdir /usr/local/lib
sudo ln -s /opt/homebrew/opt/openssl@1.1/include/openssl /usr/local/include
sudo ln -s /opt/homebrew/opt/openssl@1.1/lib/* /usr/local/lib/
```

Following [this](https://apple.stackexchange.com/questions/276307/installing-elf-on-mac-through-homebrew) place `elf.h` in `/usr/local/inclue`

##### netfliter case-sensitivity

MacOS is a case-insensitive file system, so `agnos-kernel-sdm845/net/netfliter/xt_hl.c` and `agnos-kernel-sdm845/net/netfilter/xt_HL.c` resolve as the same file, which breaks the build. To work around this, agnos development needs to take place in an "APFS (Case-sensitive)" volume. You can create one in the "Disk Utility" by clicking the "+" icon in the top bar, and selecting "APFS (Case-sensitive)" as the volume type. The resulting volume can be accessed at "/Volume/<volume-name>", and this is where you'll need to do your development.

#### Busted ideas

##### Just Build natively?

M1 is ARM-ish, so we don't actually need to cross-compile. We will need XCODE to get _some_ version of gcc/ld, and then we can look at including a specific version via LFS at a later time.

```sh
brew install openssl
# still doesn't find them :(
```

Following [this](https://apple.stackexchange.com/questions/276307/installing-elf-on-mac-through-homebrew) place `elf.h` in `/usr/local/inclue`

##### Let's build linaro GCC from source, then

[Source here](https://developer.arm.com/downloads/-/gnu-a/8-2-2018-08)

WELL, that was a bust


## Build the userspace

build:
```sh
./build_system.sh
```

load:
```sh
./flash_system.sh
```

## Build the kernel

build:
```sh
./build_kernel.sh
```

load:
```sh
# flash over fastboot
./flash_kernel.sh

# or load into running system via ssh
# ssh config needs host named 'tici'
./load_kernel.sh
```
