<!-- Recommended: Use VSCode and the extension 'Markdown All in One' to edit -->

<!-- omit in toc -->
# Setup for STM32 development on Windows with WSL2

<!-- omit in toc -->
# Table of contents

- [1. Install STM32CubeCLT - Command Line Tool](#1-install-stm32cubeclt---command-line-tool)
  - [1.1. Get the installer](#11-get-the-installer)
  - [1.2. Run the installer](#12-run-the-installer)
  - [1.3. Make the Command Line Tools available](#13-make-the-command-line-tools-available)

# 1. Install STM32CubeCLT - Command Line Tool

## 1.1. Get the installer

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

## 1.2. Run the installer

1. Run the `.sh` installer using `sudo`
   - `sudo sh <filename>.sh`
2. Accept the prompt with `y`
3. Accept location installation with `<Enter>`, which default to something similar to:
   - `/opt/st/stm32cubeclt_<version`

## 1.3. Make the Command Line Tools available

The installation will also add a script to `/etc/profile.d/` similar to `cubeclt-bin-path_<version>.sh` which will extend your `PATH`.

You do thereby have to open a new terminal for these to load in, or you source it yourself e.g.:\
`. /etc/profile.d/cubeclt-bin-path_1.16.0.sh`
