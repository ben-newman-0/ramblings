---
title: "Cross Compiling the Raspberry Pi 2 Kernel"
tags: [Cross-Compiling, Kernel, Raspberry Pi 2, Ubuntu]
---

## Cross Compiling the Raspberry Pi 2 Kernel

This is what I did to compile the kernel for the Raspberry Pi 2 using Ubuntu 14.10 (14.04 LTS used an incompatible version of gcc).

### 1. Install Dependencies

```bash
sudo apt-get install gcc-arm-linux-gnueabihf make ncurses-dev git
```

### 2. Clone Kernel Source

```bash
mkdir raspi-dev
git clone https://github.com/raspberrypi/linux --depth=1
```

### 3. Clean Build Folders

```bash
cd linux
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mrproper
```

### 4. Copy Kernel Config from Raspbian

Since the Linux kernel has hundreds of options, it's best to configure it using a working config as a starting point. Install Raspbian to a MicroSD card and boot it up on the Pi. Then run the following commands to copy the config to the pi home directory.

```bash
cd
zcat /proc/config.gz > default.config
```

Then insert the SD card back into the Ubuntu machine and copy the config to the raspi-dev directory and the Kernel build directory.

```bash
cp /media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION]/home/pi/default.config ~/raspi-dev/default.config
cp ~/raspi-dev/default.config ~/raspi-dev/linux/.config
```

### 5. Modify Kernel Configuration using menuconfig

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

### 6. Build Kernel

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
```

### 7. Install Kernel

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION] modules
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION] modules_install
```
