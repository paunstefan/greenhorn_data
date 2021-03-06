## Create Ubuntu Server automated installer
First you need to download the Ubuntu Server ISO you want to install (tested only on classic non-graphical installer).

Mount it and copy the files to a different directory

```bash
mkdir -p /mnt/iso
mount -o loop ubuntu.iso /mnt/iso

mkdir -p /opt/ubuntuiso
cp -rT /mnt/iso /opt/ubuntuiso
```

To prevent the language prompt:

```bash
cd /opt/ubuntuiso
echo en >isolinux/lang
```

Then you need a program that will create the Kickstart file (though you can do it manually if you want).

```bash
apt install system-config-kickstart
system-config-kickstart # save file to ks.cfg
```

Add a `%packages` section to the ks.cfg file where you will specify what additional software to install. My complete one looks like this:

```bash
#Generated by Kickstart Configurator
#platform=x86

#System language
lang en_US
#Language modules to install
langsupport en_US
#System keyboard
keyboard us
#System mouse
mouse
#System timezone
timezone Europe/Bucharest
#Root password
rootpw [rootpassword]
#Initial user
user --disabled
#Reboot after installation
reboot
#Use text mode install
text
#Install OS instead of upgrade
install
#Use CDROM installation media
cdrom
#System bootloader configuration
bootloader --location=mbr 
#Clear the Master Boot Record
zerombr yes
#Partition clearing information
clearpart --all --initlabel 
#System authorization infomation
auth  --useshadow  --enablemd5 
#Network information
network --bootproto=dhcp --device=eth0
#Firewall configuration
firewall --disabled 
#Do not configure the X Window System
skipx
%packages
@ ubuntu-server
openssh-server
htop
traceroute
curl
wget
git
build-essential
vim
```

Add a ''ks.preseed'' file to suppress the other options. This is what made it work for me, you can use other options:

```bash
d-i user-setup/allow-password-weak boolean true
d-i partman/confirm_write_new_label boolean true
d-i partman/confirm boolean true
d-i partman-md/confirm_nooverwrite boolean true
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman/choose_partition \
select Finish partitioning and write changes to disk
d-i partman/confirm_nooverwrite boolean true
```

In ''/isolinux/txt.cfg'', under the ''label install'' section modify the append option to look like this:

```bash
append  file=/cdrom/preseed/ubuntu-server.seed initrd=/install/initrd.gz ks=cdrom:/ks.cfg preseed/file=/cdrom/ks.preseed ---
```

In the ''/isolinux/isolinux.cfg'' file, make the timeout 10 to skip the wait at the beginning.

Now create the .iso:

```bash
mkisofs -D -r -V "AUTOINSTALL_UBUNTU" \
     -cache-inodes -J -l -b isolinux/isolinux.bin \
     -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 \
     -boot-info-table -o /opt/autoinstall.iso /opt/ubuntuiso
```

## Source

  * [Create a completety unattended install of Ubuntu](https://askubuntu.com/questions/122505/how-do-i-create-a-completely-unattended-install-of-ubuntu)