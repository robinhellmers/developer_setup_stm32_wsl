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
- [3. Install STM32CubeMX - Initialization Code Generator](#3-install-stm32cubemx---initialization-code-generator)
  - [3.1. Choose operating system](#31-choose-operating-system)
  - [3.2. Windows installation](#32-windows-installation)
  - [3.3. Linux installation](#33-linux-installation)
- [4. Initialize a project - STM32CubeMX](#4-initialize-a-project---stm32cubemx)
- [5. Setup Visual Studio Code](#5-setup-visual-studio-code)
  - [5.1. Install extensions](#51-install-extensions)
    - [5.1.1. C/C++ extension](#511-cc-extension)
    - [5.1.2. Build system extension](#512-build-system-extension)
  - [5.2. Extensions configurations](#52-extensions-configurations)
    - [5.2.1. C/C++](#521-cc)
    - [5.2.2. Makefile Tools](#522-makefile-tools)

# 1. Pre-requisites

- WSL2 installed with an Ubuntu instance
- Visual Studio Code (VSCode) installed
- VSCode 'Remote Development' extension installed for access to the WSL instance

# 2. Install STM32CubeCLT - Command Line Tool

## 2.1. Get the installer

1. Download the [**Generic Linux Installer**](
https://www.st.com/en/development-tools/stm32cubeclt.html), instead of the
Debian based one
   - It requires an account at ST
2. Go to the **Windows explorer** under **Downloads**, where the downloaded
`.zip` is located
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
3. Accept location installation with `<Enter>`, which default to something
similar to:
   - `/opt/st/stm32cubeclt_<version`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.3. Make the Command Line Tools available

The installation will also add a script to `/etc/profile.d/` similar to
`cubeclt-bin-path_<version>.sh` which will extend your `PATH`.

You do thereby have to open a new terminal for these to load in, or you source
it yourself e.g.:\
`. /etc/profile.d/cubeclt-bin-path_1.16.0.sh`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 2.4. Missing packages

### 2.4.1. Find out missing packages

- Check the [**Release Note at the download page**](
https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), under
**Installing on Linux** which describes the missing packages as well as you
needing the correct python version.

- You can use the `ldd <executable>` command to see dependencies\
| *prints the shared objects (shared libraries) required by each program or
shared object*

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

These are not even available in the `apt` `universe` repository for newer Ubuntu
versions. Which is why we install them through the Ubuntu archive.

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

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

# 3. Install STM32CubeMX - Initialization Code Generator

## 3.1. Choose operating system

From testing both an install on Windows as well as on an Ubuntu WSL2 instance,
I would recommend the Windows installation to avoid issues. This is a GUI based
tool and thereby works the best on Windows as that is the host system.

It works great with generating code within the WSL2 Ubuntu instance.

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 3.2. Windows installation

1. Download the [**Windows Installer**](
https://www.st.com/en/development-tools/stm32cubeclt.html)
   - It requires an account at ST
2. Unzip the `.zip` in Windows
3. Execute the `.exe` file
4. Follow the installation instructions

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 3.3. Linux installation

[Upcoming link to markdown file with instructions](.)

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

# 4. Initialize a project - STM32CubeMX

Create a project which builds with Makefile. It shall be created to
demonstrate a **Intellisense** setup for C/C++ with Makefile.

1. Start **STM32CubeMX** on Windows.
2. Click **Start My project from ST Board**
    - Wait while it downloads information

    <p align="center">
        <img
            src="https://github.com/user-attachments/assets/4dcfb8a0-be67-4b1a-b00b-84536c3d5781"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="40%"
        />
    </p>

3. Under **Commercial Part Number**, search for `P-NUCLEO-WB55-NUCLEO`

    <p align="center">
        <img src="https://github.com/user-attachments/assets/6b4c6cab-2918-4130-9694-36256d96640b"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="40%"
        />
    </p>

4. Double-click the found board with the same commercial part number

    <p align="center">
        <img src="https://github.com/user-attachments/assets/bdb0c5aa-162a-4997-9be3-c909921bccb6"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="40%"
        />
    </p>

5. In the **Board Project Options** popup window, select
   `Generate demonstration code` and then click **OK**

    <p align="center">
        <img src="https://github.com/user-attachments/assets/d1ab84c0-7077-44f8-917b-92716a814984"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="40%"
        />
    </p>

6. Click the **Project Manager** tab.

    <p align="center">
        <img src="https://github.com/user-attachments/assets/cabecb64-4bd7-4d1e-816c-e7070a209637"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="90%"
        />
    </p>

7. Enter the fields:
    - `Project Name`
        - Which will be the directory name as well. Avoid spaces as it is in the
linux space
    - `Project Location`
        - Reach the WSL2 Ubuntu instance throuch `\\wsl.localhost\<wsl_instance>\`
        - Can e.g. be reached through the Windows Explorer
        - Find WSL instance name by opening **CMD** and listing all instances
`wsl --list`
        - Example location: `\\wsl.localhost\Ubuntu-24.04-6\home\hellmers\git\project`
    - `Application Structure`
        - Not as important, choose `Advanced`
    - `Toolchain Folder Location`
        - Automatically filled using `Project Location` and `Project Name`
    - `Toolchain / IDE`
        - Select `Makefile` as this guide showcases this
        - Could as well have been `CMake`

    <p align="center">
        <img src="https://github.com/user-attachments/assets/ab2805f3-18e7-4f83-bf57-28bc383e4e3a"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="90%"
        />
    </p>

8. Click the **GENERATE CODE** button, top right above the tabs
    - Wait a little while for the code to generate

    <p align="center">
        <img src="https://github.com/user-attachments/assets/12ceb8e2-266f-4efa-8582-65a900689824"
            alt="ACCESS TO BOARD SELECTOR button in CubeMX GUI"
            width="30%"
        />
    </p>

9. Open the WSL2 Ubuntu instance and go to the project location
10. Install `make` with `apt`
    - `sudo apt install make`
11. While in the project directory, where the generated `Makefile` is, compile everything:
    - `make`
    - Just to make sure there are no issues

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

# 5. Setup Visual Studio Code

## 5.1. Install extensions

Install the Visual Studio Code (VSCode) extensions e.g. through the GUI
Extensions tab.

### 5.1.1. C/C++ extension

Install the extension:

- C/C++ extension
  - ID: `ms-vscode.cpptools`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

### 5.1.2. Build system extension

The build system extension of course depends on which build system you have
chosen to use when e.g. generating a project with STM32CubeMX.

Example build systems:

- Makefile
- CMake

This guide specifically used Makefile.

Install your corresponding extension:

- Makefile Tools
  - ID: `ms-vscode.makefile-tools`
- CMake Tools
  - ID: `ms-vscode.cmake-tools`

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

## 5.2. Extensions configurations

Open VSCode at the project root using `code .` if you are in the project root.

### 5.2.1. C/C++

1. Open the `Command Palette`:
    - Press `Ctrl` + `Shift` + `P`
2. Enter `C/C++: Edit Configurations (UI)`
    - Creates a configuration file at the project directory `.vscode/c_cpp_properties.json`
    - Presents configuration options
3. Configure the following fields:
    1. `Compiler path`
        1. Check the drop-down list if it presents you with the full path to `arm-none-eabi-gcc`
        2. If not, open the Ubuntu instance and check the path with
            - `which arm-none-eabi-gcc`
    2. `IntelliSense mode`
        - Select `linux-gcc-arm`
    3. `Include path`
        - Should already have `${workspaceFolder}/**`
4. At the bottom, expand the **Advanced Settings** drop-down
5. Configure the following fields:
    1. `Configuration provider`j
        - Tells which build system that is used and which extension to integrate
with
        - For Makefile: `ms-vscode.makefile-tools`
        - Extension ID can be found by:
            1. Open **Extensions** by pressing `Ctrl` + `Shift` + `X`
            2. Click the cogwheel to the right of your build system extension
                - E.g. `Makefile Tools`
            3. Click **Copy Extension ID**

---

This results in a similar  `.vscode/c_cpp_properties.json`:

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "compilerPath": "/opt/st/stm32cubeclt_1.16.0/GNU-tools-for-STM32/bin/arm-none-eabi-gcc",
            "cStandard": "c17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "linux-gcc-arm",
            "configurationProvider": "ms-vscode.makefile-tools"
        }
    ],
    "version": 4
}
```

<div align="right">
  <a href="#table-of-contents">Back to TOC</a>
</div>

### 5.2.2. Makefile Tools

For **IntelliSense** to work properly, it needs to read the commands executed
when running make. To do this, it runs something similar to `make clean`
followed by `make --dry-run`, basically not executing the commands but printing
out the commands to be executed. That output is put into a temporary file, which
then is read and used for IntelliSense.

First, try with just:

1. Open the `Command Palette`:
    - Press `Ctrl` + `Shift` + `P`
2. Enter `Makefile: Configure`

---

If the output in the `Makefile tools` terminal or `.vscode/extension.log` shows
failed execution, you can do it manually:

1. Open the `Command Palette`:
    - Press `Ctrl` + `Shift` + `P`
2. Enter `Makefile: Open Build Log Setting`
3. Configure the following field:
    - `Build Log`
        - Enter `./dryrun.log`
4. Generate `dryrun.log`
    1. Open the project location
    2. Clean up any previous build
        - `make clean`
    3. Generate the log:
        - `make --dry-run > dryrun.log`
5. Reload the window
6. Potentially run the same as previous:
    - `Makefile: Configure`

---

This results in the following setting in `.vscode/settings.json`:

```json
{
    "makefile.buildLog": "./dryrun.log"
}
```
