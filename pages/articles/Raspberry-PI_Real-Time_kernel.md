# Raspberry Pi real-time kernel

## Prerequisites

First you need to get the GCC toolchain for the Raspberry Pi from <https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads>, you need the `arm-none-linux-gnueabihf` version. Unpack it and put it somewere where you can access it.

Then clone the Raspberry Pi kernel repo:

```
git clone https://github.com/raspberrypi/linux.git
```

And switch to the most recent RT branch (`rpi-4.19.y-rt` at the time of writing).

## Compilation

```
# for RPi 2/3
cd linux
KERNEL=kernel7
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig

# for RPi 0
cd linux
KERNEL=kernel
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig
```

As the `CROSS_COMPILE` parameter put the path of the `bin` directory in the previously downloaded toolchain. For example: `/opt/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/bin/arm-none-linux-gnueabihf-`

The `KERNEL` variable will name the kernel image, it's not really important if you force the device to boot a certain image. Otherwise, each Pi version will look for certain default kernel names.

Now run `make menuconfig` to enable `CONFIG_PREEMPT_RT_FULL` if it is not already enabled.

For the actual compilation run the following:

```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs
```

Now install the modules and device tree files:

```
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- INSTALL_MOD_PATH=[install_dir] INSTALL_DTBS_PATH=[install_dir] modules_install dtbs_install
```

Move the image and rename it so it's differentiated from the already existing kernel.

```
mkdir [install_dir]/boot
./scripts/mkknlimg ./arch/arm/boot/zImage [install_dir]/boot/$KERNEL.img

cd [install_dir]/boot
mv [kernel_name].img [kernel_name]-rt.img 
```

Finally, archive everything.

```
cd [install_dir]
tar czf ../rt-kernel.tgz *
```

## Install the kernel on the Raspberry Pi

Copy the archive to the Pi:
```
scp rt-kernel.tgz pi@<ipaddress>:/tmp
```

Then unarchive and copy the files to their final location:
```
cd /tmp
tar xzf rt-kernel.tgz
cd boot
sudo cp -rd * /boot/
cd ../lib
sudo cp -dr * /lib/
cd ../overlays
sudo cp -d * /boot/overlays
cd ..
sudo cp -d bcm* /boot/
```

Finally, edit the `/boot/config.txt` by adding the following line:
```
kernel=[kernel_name]-rt.img
```

Now reboot the device and the kernel is installed.

### Sources

 * <https://www.raspberrypi.com/documentation/computers/linux_kernel.html>
 * <https://elinux.org/Raspberry_Pi_Kernel_Compilation#2._Cross_compiling_from_Linux>
 * <https://lemariva.com/blog/2018/07/raspberry-pi-preempt-rt-patching-tutorial-for-kernel-4-14-y>