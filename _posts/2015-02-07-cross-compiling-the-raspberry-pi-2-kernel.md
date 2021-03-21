---
title: "Cross Compiling the Raspberry Pi 2 Kernel"
tags: [Cross-Compiling, Kernel, Raspberry Pi 2, Ubuntu]
---

This is what I did to compile the kernel for the Raspberry Pi 2 using Ubuntu 14.10 (14.04 LTS used an incompatible version of gcc).
<h2>1. Install Dependencies</h2>
<pre>sudo apt-get install gcc-arm-linux-gnueabihf make ncurses-dev git</pre>
<h2>2. Clone Kernel Source</h2>
<pre>mkdir raspi-dev
git clone https://github.com/raspberrypi/linux --depth=1</pre>
<h2>3. Clean Build Folders</h2>
<pre>cd linux
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mrproper</pre>
<h2>4. Copy Kernel Config from Raspbian</h2>
Since the Linux kernel has hundreds of options, it's best to configure it using a working config as a starting point. Install Raspbian to a MicroSD card and boot it up on the Pi. Then run the following commands to copy the config to the pi home directory.
<pre>cd
zcat /proc/config.gz &gt; default.config</pre>
Then insert the SD card back into the Ubuntu machine and copy the config to the raspi-dev directory and the Kernel build directory.
<pre>cp /media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION]/home/pi/default.config ~/raspi-dev/default.config
cp ~/raspi-dev/default.config ~/raspi-dev/linux/.config</pre>
<h2>5. Modify Kernel Configuration using menuconfig</h2>
<pre>make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig</pre>
<h2>6. Build Kernel</h2>
<pre>make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-</pre>
<h2>7. Install Kernel</h2>
<pre>make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION] modules
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=media/[Ubuntu Username]/[GUID OF SD CARDs EXT4 PARTITION] modules_install</pre>
