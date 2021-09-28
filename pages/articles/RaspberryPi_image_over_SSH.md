# Raspberry Pi copy system image over SSH

First, on the Raspberry Pi you must check what is the name of the storage device.
```bash
lsblk -p

# For example it could be /dev/mmcblk0
```

Then, on the machine you want to save the image to, run the following command:
```bash
ssh (-p [p_nr]) username@host sudo dd if=/dev/mmcblk0 bs=1M | pv | gzip -c > file.img.gz
```