# Backup files with rsync

Rsync can be use to make incremental backups of files and directories on the same system or over the network.

To copy the contents of a directory to another directory on the same machine, run the follwing:

```bash
rsync -av --delete dir1/ dir2/

# the trailing slash means that you only transfer the contents, not the directory itself (remove it if you want otherwise)
# the delete option is to delete any items that are in dir2 but not in dir1
```

To transfer between systems over the network it's almost as simple:

```
# push
rsync -av --delete dir1/ username@remote_host:destination_directory

# pull
rsync -av --delete username@remote_host:/home/username/dir1 place_to_sync_on_local_machine
```