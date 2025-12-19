---
Title: mergerfs
Description: Guide to setting up and using mergerfs for pooling multiple drives into a single mount point.
---


# Overview
[Github](https://github.com/trapexit/mergerfs?tab=readme-ov-file)

[Official Docs](https://trapexit.github.io/mergerfs/latest/)

----
**mergerfs** is a [union filesystem](https://en.wikipedia.org/wiki/Union_mount) that makes multiple storage devices or filesystems appear as a single unified directory. Built on [FUSE (Filesystem in Userspace)](https://en.wikipedia.org/wiki/Filesystem_in_Userspace), it is designed to simplify how you manage files across several independent filesystems without the complexity, fragility, or cost of RAID or similar storage aggregation technologies.

Think of mergerfs as a smart pooling layer: you can combine any number of [existing filesystems](https://trapexit.github.io/mergerfs/latest/faq/usage_and_functionality/#can-mergerfs-be-used-with-filesystems-which-already-have-data) — whether they are on hard drives, SSDs, network shares, or other mounted storage — into what looks like one large filesystem, while still maintaining direct access to each individual filesystem. Unlike RAID, there's no rebuild process if a device fails. You only lose the files that were on that specific filesystem. You can also add or remove filesystems at any time without restructuring your entire pool.

# Usage & Branch Setup
----
> [!WARNING] Important
> Never write data directly to the `/mnt/disk1` folders if you can help it. Always write to `/mnt/storage`. If you modify files on the backing disks directly while MergerFS is running, you might see cache inconsistencies until you remount.
## Mount points
[Branches Docs](https://trapexit.github.io/mergerfs/latest/config/branches/)
### Naming Convention 
**HDDs:** `/mnt/hdd`
**SATA SSDs:** `/mnt/ssd`
**NVME/M2:** `/mnt/nvme`
**REMOTE:** `/mnt/remote`
##### Example
```bash 
$ ls -lh /mnt/
total 16K
drwxr-xr-x 8 root root 4.0K Aug 18  2024 hdd
drwxr-xr-x 6 root root 4.0K Oct  8  2024 nvme
drwxr-xr-x 3 root root 4.0K Aug 24  2024 remote
drwxr-xr-x 3 root root 4.0K Jul 14  2024 ssd

$ ls -lh /mnt/hdd/
total 8K
d--------- 2 root root 4.0K Apr 14 15:58 10T-01234567
d--------- 2 root root 4.0K Apr 12 20:51 20T-12345678

$ ls -lh /mnt/nvme/
total 8K
d--------- 2 root root 4.0K Apr 14 16:00 1T-ABCDEFGH
d--------- 2 root root 4.0K Apr 14 23:24 1T-BCDEFGHI

$ ls -lh /mnt/remote/
total 8K
d--------- 2 root root 4.0K Apr 12 20:23 foo-sshfs
d--------- 2 root root 4.0K Apr 12 20:24 bar-nfs


# You can find the serial number of a drive using lsblk
$ lsblk -d -o NAME,PATH,SIZE,SERIAL
NAME    PATH           SIZE SERIAL
sda     /dev/sda       9.1T 01234567
sdb     /dev/sdb      18.2T 12345678
nvme0n1 /dev/nvme0n1 953.9G ABCDEFGH
nvme1n1 /dev/nvme1n1 953.9G BCDEFGHI
```

### Permissions and mode
To ensure the directory is **only** used as a point to mount another filesystem it is good to lock it down as much as possible. Be sure to do this **before** mounting a filesystem to it.

```bash
$ sudo chown root:root /mnt/hdd/10T-XYZ
$ sudo chmod 0000 /mnt/hdd/10T-XYZ
$ sudo setfattr -n user.mergerfs.branch_mounts_here
```

### Pool Creation
>[!info] For Linux v6.6 and above (defaults - quick start)
>- cache.files=off
>- category.create=pfrd
>- func.getattr=newest
>- dropcacheonclose=false

**Create directory for pool**
```bash
sudo mkdir -p /mnt/storage
```

**Run pool setup from CLI***
```bash
	mergerfs -o cache.files=off,category.create=mspmfs,func.getattr=newest,dropcacheonclose=false,minfreespace=200G,moveonenospc=true /mnt/disk*/mnt /mnt/storage
```

**Add to /etc/fstab for pool persistance**
```bash
# To be used - defaults,  Fill disk w/ exist path then move to next disk, keep 200GB free, move drives if file to be written >= 200GB
# Syntax: <branches> <mountpoint> <type> <options> <dump> <pass>

### CURRENT /etc/fstab FOR MERGER FS 
/mnt/hdd/WD-B00WHLZD:/mnt/hdd/WD-B00WUHXD /mnt/storage mergerfs cache.files=off,category.create=mspmfs,func.getattr=newest,dropcacheonclose=false,minfreespace=200G,moveonenospc=true,fsname=mergerfs 0 0


```

**Mount the Pool & verify**
```bash
sudo mount -a
```bash
# verify syntax 
sudo findmnt --verify

# reload fstab
sudo systemctl daemon-reload

df -h
```

**Docker Integration** 
**Do NOT use:** `/mnt/disk1/media`
**USE:** `/mnt/storage/media`

Example `docker-compose.yml`:
```yaml
services:
  plex:
    image: linuxserver/plex
    volumes:
      - /mnt/storage/media/movies:/movies
      - /mnt/storage/media/tv:/tv
```

### Handling Deleted Files
---
When you delete a file from a MergerFS pool, it **only removes it from the original drive**. If a file appears to still exist, try refreshing the filesystem cache:

```bash
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

### Tips and Notes
-  Run mergerfs as `root`. mergerfs is designed and intended to be run as `root` and may exhibit incorrect behavior if run otherwise.

- If you do not see some directories and files you expect, policies seem to skip branches, you get strange permission errors, etc. be sure the underlying filesystems' permissions are all the same. Use `mergerfs.fsck` to audit the filesystem for out of sync permissions.

- [Kodi](http://kodi.tv/), [Plex](http://plex.tv/), [Subsonic](http://subsonic.org/), etc. can use directory [mtime](http://linux.die.net/man/2/stat) to more efficiently determine whether to scan for new content rather than simply performing a full scan. If using the default `getattr` policy of `ff` it's possible those programs will miss an update on account of it returning the first directory found's `stat` info and it is a later directory on another mount which had the `mtime` recently updated. To fix this you will want to set `func.getattr=newest`. Remember though that this is just `stat`. If the file is later `open`'ed or `unlink`'ed and the policy is different for those then a completely different file or directory could be acted on.
