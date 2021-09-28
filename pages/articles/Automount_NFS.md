## Automount NFS share

First you need to install `nfs-common` and `autofs` if your distro doesn't have them by default.

```bash
sudo apt-get install nfs-common
sudo apt-get install autofs
```

Next, modify the `/etc/auto.master` file. Add the following line:

```
<mount point>  /etc/auto.nfs --timeout=0 --ghost
```

`/etc/auto.nsf` will be the map file(you can call it however you want), the timeout configures the time in seconds after which the share will be unmounted, 0 means never unmount, ghost creates placeholder directories for the mounts.

Now you need to create the map file (/etc/auto.nfs) and add the line:

```
<mount_directory> -fstype=nfs,rw,async,vers=3 <server_address>:<server share location>
```

Using NFSv3 because v4 still has some bugs. Async will greatly improve transfer speed.

Finally, restart and enable the autofs service.

```
sudo systemctl start autofs.service
sudo systemctl enable autofs.service
```