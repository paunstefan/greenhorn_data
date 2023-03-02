# Raspberry Pi 3 U-Boot

This is a tutorial on how to boot a Linux system on a Raspberry Pi 3 using U-boot.

## Prepare the SD card

To boot Linux on a RPi you need a boot partition containing the bootloader and the Linux kernel, and a partition
with a root filesystem.

Insert an SD card and find its device file using `lsblk` or `dmesg`. Delete the existing partitions if they exist
and create a new 100MB boot partition and a root partition on the remaining space. Then create a FAT32 filesystem
on the boot partition and mount it. (We'll come back to the root filesystem later)

```
sudo mkfs.vfat -F 32 -n boot /dev/sdc1
sudo mount /dev/sdc1 /mnt/boot
```

## The toolchain

If you are not using a Raspberry Pi to build everything you need the specific toolchain to be able to build
for it. You can manually download the one you need, or use `crosstool-NG` to automate everything (and will work
for most architectures).

```
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng

./bootstrap
./configure --prefix=${PWD}
make
make install
export PATH="${PWD}/bin:${PATH}"
```

This installed `crosstool-NG`, now you need to build the toolchain using it. Use `ct-ng list-samples` to list the available architectures
and `ct-ng menuconfig` after choosing one to modify it. I will choose `aarch64-rpi3-linux-gnu`.

```
ct-ng aarch64-rpi3-linux-gnu
ct-ng build
export PATH=~/x-tools/aarch64-rpi3-linux-gnu/bin:$PATH
export CROSS_COMPILE=aarch64-rpi3-linux-gnu-
```

## U-boot

Download the U-boot repo and build it:

```
git clone git://git.denx.de/u-boot.git
cd u-boot
make ARCH=arm CROSS_COMPILE=aarch64-rpi3-linux-gnu- rpi_arm64_defconfig
make ARCH=arm CROSS_COMPILE=aarch64-rpi3-linux-gnu-
sudo cp u-boot.bin /mnt/boot
```

## Linux

Download the Linux kernel(preferably from the Raspberry Pi repo) and build it.

```
git clone --depth=1 -b rpi-5.10.y https://github.com/raspberrypi/linux.git
cd linux
make ARCH=arm64 CROSS_COMPILE=aarch64-rpi3-linux-gnu- bcmrpi3_defconfig
make -j4 ARCH=arm64 CROSS_COMPILE=aarch64-rpi3-linux-gnu-
```

Then copy the image and device tree files:

```
cd arch/arm64/boot
sudo cp Image /mnt/boot
sudo cp dts/broadcom/bcm2710-rpi-3-b.dtb /mnt/boot/
```

## Raspberry Pi firmware

The Raspberry Pi uses a proprietary bootloader that needs to be present on the SD card, which will in turn load the U-boot image.

```
wget https://github.com/raspberrypi/firmware/raw/master/boot/bootcode.bin
wget https://github.com/raspberrypi/firmware/raw/master/boot/start.elf

cat << EOF > config.txt
enable_uart=1
arm_64bit=1
kernel=u-boot.bin
EOF

sudo cp  bootcode.bin config.txt start.elf /mnt/boot
```

## Buildroot

You could build a root filesystem manually and make it into a CPIO archive, but a easier and more flexible method it to
use Buildroot (which can actually also build a complete bootable SD card image).

```
wget https://buildroot.org/downloads/buildroot-2022.11.tar.xz
tar -xf buildroot-2020.02.8.tar.gz
cd buildroot-2020.02.8
make raspberrypi3_64_defconfig
make menuconfig
# here go to the Kernel menu and unselect it so it will only build the root FS

make
```

Now you have the root filesystem, write it to the SD card partition:

```
sudo dd if=output/images/rootfs.ext4 of=/dev/sdc2
```

## Final steps

Now you can remove the SD card from the PC and insert it into the Raspberry Pi. Plug an adapter into the UART pins
and connect to it from a terminal emulator, such as Minicom.

You will be greeted by the U-boot command line. Enter the following boot options:

```
setenv bootcmd 'fatload mmc 0 $kernel_addr_r Image ; fatload mmc 0:1 $fdt_addr bcm2710-rpi-3-b.dtb ; booti $kernel_addr_r - $fdt_addr'
setenv bootargs 8250.nr_uarts=1 console=ttyS0,115200 root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait init=/bin/sh
saveenv
```

Restart the device. You should now be successfully booted into Linux.
