# Minimal Linux System

In this article I will describe how to build the simplest Linux system,
for when you think distros are bloat :).

Dependencies:
* build-essential (Linux C toolchain, it is called differently on different distros)
* bison
* flex
* git
* libelf
* libssl
* ncurses
* qemu

Before we start, create a workspace where you will download the sources and build everything.

```bash
mkdir linux
cd linux
mkdir build
export BASE=$(pwd)
```

## Linux kernel

The first thing you need to do, of course, is to build the Linux kernel. You can download it from here <https://cdn.kernel.org/pub/linux/kernel/>, just choose a version you like,
I downloaded `5.4.189`.

```bash
curl https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.189.tar.xz | tar xJf -
```

In the downloaded directory, you must first build the config, I chose `x86_64_defconfig`, which is
pretty big, but it should cover anything you need for a working system.

Then start the kernel build.

```bash
make O=$BASE/build/linux-x86-basic x86_64_defconfig
make O=$BASE/build/linux-x86-basic -j$(nproc)
```

The `$(nproc)` will return the number of cores on your system so the build is parallelized.

This is all, in `$(BASE)/build/linux-x86-basic` there should the newly created kernel.

## Busybox

We'll come back to the kernel, but now it's time to build the userspace program. I will use
Busybox, which contains most Unix tools in a single executable.

It can be found at <https://busybox.net/downloads/>.

```bash
curl https://busybox.net/downloads/busybox-1.35.0.tar.bz2 | tar xjf -
```

In the new directory, first configure Busybox.

```bash
mkdir $BASE/build/busybox-x86 # it does not create the directory like the kernel
make O=$BASE/build/busybox-x86 defconfig
```

To remove the need for shared libs, build Busybox as a static binary.

```bash
make O=$BASE/build/busybox-x86 menuconfig

# and search and select "Build BusyBox as a static binary (no shared libs)"
```

To build it:

```bash
cd $BASE/build/busybox-x86
make -j$(nproc)
make install
```

Everything should now be in the `_install` directory.

### Initramfs

Initramfs is the filesystem the kernel first loads, and contains basic programs and drivers
used to bootstrap the actual filesystem, but here we will use this as the actual 
system. This is the newer feature that creates a virtal filesystem directly in RAM.
The previous version was called `initrd`, which created a ramdisk, in which it loaded 
a real filesystem.

Create a new directory in the `build` directory, create the filesystem skeleton, and copy the 
previously built files.

```bash
mkdir -p $BASE/build/initramfs/busybox-x86/{bin,sbin,etc,proc,sys,usr/{bin,sbin}}
cp -av $BASE/build//busybox-x86/_install/* $BASE/build/initramfs/busybox-x86
```

When the kernel boots, it searches for an `init` file to run, which will be PID 1. 
We need to create it.

```bash
cat << EOT >> $BASE/build/initramfs/busybox-x86/init
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys

exec /bin/sh
EOT

chmod +x $BASE/build/initramfs/busybox-x86/init
```

The format the kernel uses for the initramfs is a CPIO archive, you can make it with
the following commands:

```bash
find . -print0 \
| cpio --null -ov --format=newc \
| gzip > $BASE/build/initramfs-busybox-x86.cpio.gz
```
(The `-print0` option adds a NULL character after file names,
the `--null` option for CPIO makes it take into consideration that character)

### Booting the system

Everything is now done, you can use QEMU to run the newly built Linux system:

```bash
qemu-system-x86_64 \
    -kernel $BASE/build/linux-x86-basic/arch/x86_64/boot/bzImage \
    -initrd $BASE/build/initramfs-busybox-x86.cpio.gz \
    -nographic -append "console=ttyS0"
```

After it boots, you should have a prompt:

```bash
/ # uname -a
Linux (none) 5.4.189 #1 SMP Mon May 9 19:09:19 EEST 2022 x86_64 GNU/Linux
```

## No Busybox init (1)

If you want to also get rid of Busybox, such as for embedded platforms where you only 
want to run your program, you can replace Busybox with your own executable.

```bash
mkdir $BASE/build/initramfs/init_c
touch main.c
```

In the new `main.c` file, add you code.

```c
#include <stdio.h>

int main(){
    printf("Hello world from C\n");
    sleep(999999999);
}
```

Make it into a CPIO archive.

```bash
find . | cpio -o -H newc | gzip > $BASE/build/init_c.cpio.gz
```

Now it can be run:

```bash
qemu-system-x86_64 \
    -kernel $BASE/build/linux-x86-basic/arch/x86_64/boot/bzImage \
    -initrd $BASE/build/init_c.cpio.gz \
    -nographic -append "console=ttyS0"
```

## No Busybox init (2)

Lets now do it in Rust. Create a new rust project `cargo new init`, and modify the `main.c` file:

```rust
fn main() {
    println!("Hello, world from Rust!");

    let forever = core::time::Duration::from_secs(999999999);
    std::thread::sleep(forever);
}
```

(This is not really needed, but otherwise the program will exit and the kernel will panic)

You need to make a binary that is statically linked to the C standard library:

```bash
RUSTFLAGS="-C target-feature=+crt-static" cargo build --release
```

Make the CPIO archive now:

```bash
mkdir cpio_dir
cd cpio_dir
cp ../target/release/init .     # these are here because the build dir also has other stuff

find . | cpio -o -H newc | gzip > $BASE/build/init_rust.cpio.gz
```

And finally run:

```bash
qemu-system-x86_64 \
    -kernel $BASE/build/linux-x86-basic/arch/x86_64/boot/bzImage \
    -initrd $BASE/build/init_rust.cpio.gz \
    -nographic -append "console=ttyS0"
```

### Extra Rust

For a more C-like experience, you can directly use `rustc`, just like I used `GCC` previously.

```bash
rustc -o init -C target-feature=+crt-static main.rs
```

After this, you will have the `init` file, without using Cargo.

## Sources

* <https://ops.tips/notes/booting-linux-on-qemu/>
* <https://mgalgs.io/2015/05/16/how-to-build-a-custom-linux-kernel-for-qemu-2015-edition.html>
* [Building the Simplest Possible Linux System - Rob Landley](https://www.youtube.com/watch?v=Sk9TatW9ino)