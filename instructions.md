<!-- Recommended: Use VSCode and the extension 'Markdown All in One' to edit -->

<!-- omit in toc -->
# Setup for STM32 development on Windows with WSL2

<!-- omit in toc -->
# Table of contents

- [1. Pre-requisites](#1-pre-requisites)
- [2. Install STM32CubeCLT - Command Line Tool](#2-install-stm32cubeclt---command-line-tool)
  - [2.1. Get the installer](#21-get-the-installer)
  - [2.2. Run the installer](#22-run-the-installer)
  - [2.3. Make the Command Line Tools available](#23-make-the-command-line-tools-available)
  - [2.4. Missing packages](#24-missing-packages)
    - [2.4.1. Find out missing packages](#241-find-out-missing-packages)
    - [2.4.2. Install missing packages](#242-install-missing-packages)
      - [2.4.2.1. `libtinfo.so.5` - Install package `libtinfo5`](#2421-libtinfoso5---install-package-libtinfo5)
      - [2.4.2.2. `libncurses.so.5` - Install package `libncurses5`](#2422-libncursesso5---install-package-libncurses5)
      - [2.4.2.3. Verify the recognition of the new packages](#2423-verify-the-recognition-of-the-new-packages)
  - [2.5. Verify that `arm-none-eabi-gdb` is executable](#25-verify-that-arm-none-eabi-gdb-is-executable)

# 1. Pre-requisites

- WSL2 installed with an Ubuntu instance

# 2. Install STM32CubeCLT - Command Line Tool

## 2.1. Get the installer

1. Download the [**Generic Linux Installer**](https://www.st.com/en/development-tools/stm32cubeclt.html), instead of the Debian based one
   - It requires an account at ST
2. Go to the **Windows explorer** under **Downloads**, where the downloaded `.zip` is located
3. Copy the Windows path from the file explorer
   - E.g. `C:\Users\<user>\Downloads`
4. Open your WSL Ubuntu instance
5. Get the linux style path
   - `wslpath "<path>"`
       - Observe the quotation
   - E.g. `wslpath "C:\Users\<user>\Downloads"`
6. Move the `.zip` installer to your home directory
   - `mv <path>/<file> ~`
7. Install `unzip`
   - `sudo apt install unzip`
8. Unzip the installer
   - `cd ~`
   - `unzip <file>`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.2. Run the installer

1. Run the `.sh` installer using `sudo`
   - `sudo sh <filename>.sh`
2. Accept the prompt with `y`
3. Accept location installation with `<Enter>`, which default to something similar to:
   - `/opt/st/stm32cubeclt_<version`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.3. Make the Command Line Tools available

The installation will also add a script to `/etc/profile.d/` similar to `cubeclt-bin-path_<version>.sh` which will extend your `PATH`.

You do thereby have to open a new terminal for these to load in, or you source it yourself e.g.:\
`. /etc/profile.d/cubeclt-bin-path_1.16.0.sh`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.4. Missing packages

### 2.4.1. Find out missing packages

* Check the [**Release Note at the download page**](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), under **Installing on Linux** which describes the missing packages as well as you needing the correct python version.

* You can use the `ldd <executable>` command to see dependencies\
| *prints the shared objects (shared libraries) required by each program or shared object*

Find out missing packages for `gdb` installed with STM32CubeCLT.

`ldd "$(which arm-none-eabi-gdb)" | grep 'not found'`

Probably outputs:

```text
        libncurses.so.5 => not found
        libtinfo.so.5 => not found
```

Which also probably reflects if you try to run it:

```shell
arm-none-eabi-gdb --version
```

With e.g. the output:

```text
arm-none-eabi-gdb: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory
```

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

### 2.4.2. Install missing packages

`libncurses.so.5` depends on `libtinfo.so.5`, which is why we start installing `libtinfo`.

These are not even available in the `apt` `universe` repository for newer Ubuntu versions. Which is why we install them through the Ubuntu archive.

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

---

#### 2.4.2.1. `libtinfo.so.5` - Install package `libtinfo5`

Download the `.deb` file:

```shell
wget \
    http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.4-2_amd64.deb
```

Install using the file:

```shell
sudo dpkg -i ./libtinfo5_6.4-2_amd64.deb
```

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

---

#### 2.4.2.2. `libncurses.so.5` - Install package `libncurses5`


Download the `.deb` file:

```shell
wget \
    http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libncurses5_6.4-2_amd64.deb
```

Install using the file:

```shell
sudo dpkg -i ./libncurses5_6.4-2_amd64.deb
```

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

---

#### 2.4.2.3. Verify the recognition of the new packages

```shell
ldd "$(which arm-none-eabi-gdb)"
```

Which outputs something similar to:

```text
        linux-vdso.so.1 (0x00007ffec9943000)
        libncurses.so.5 => /lib/x86_64-linux-gnu/libncurses.so.5 (0x00007f7d3ee6f000)
        libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007f7d3ee3f000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f7d3ee3a000)
        libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f7d3df83000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f7d3ed51000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f7d3ed22000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f7d3ed1d000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7d3dd71000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f7d3ee9c000)
```

Indicating that both `libncurses.so.5` and `libtinfo.so.5` are found.

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.5. Verify that `arm-none-eabi-gdb` is executable

```shell
arm-none-eabi-gdb --version
```

Which should not complain and output something similar to:

```text
GNU gdb (GNU Tools for STM32 12.3.rel1.20240612-1315) 13.2.90.20230627-git
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
