---
tags:
  - snapraid
  - storage
  - backup
  - mergerfs
  - nas
---
https://www.snapraid.it/ 
[Snapraid AIO Script](https://github.com/auanasgheps/snapraid-aio-script?tab=readme-ov-file)
---
# Install and Compile SnapRAID (latest version)
```bash
wget https://github.com/amadvance/snapraid/releases/download/v13.0/snapraid-13.0.tar.gz

tar xzvf snapraid-13.0.tar.gz

cd snapraid

./autogen.sh
./configure
make
make check
sudo make install

#check running version
snapraid -V


```

# Overview
---
To use SnapRAID, you need to first select one disk in your disk array to dedicate to "**parity**" information. With one disk for parity, you will be able to recover from a single disk failure, similar to RAID5.

If you want to recover from more disk failures, similar to RAID6, you must reserve additional disks for **parity**. Each additional parity disk allows recovery from one more disk failure.

As parity disks, you must pick the largest disks in the array, as the parity information may grow to the size of the largest data disk in the array.

These disks will be dedicated to storing the "**parity**" files. You should not store your data on them.

Then, you must define the "**data**" disks that you want to protect with SnapRAID. The protection is more effective if these disks contain data that rarely changes. For this reason, **it's better to NOT include the Windows C:\ disk or the Unix /home, /var, and /tmp directories.**

The list of files is saved in the "**content**" files, usually stored on the data, parity, or boot disks. This file **contains the details of your backup**, **including all the checksums to verify its integrity**. The "**content**" file is stored in multiple copies, and each copy must be on a different disk to ensure that, even in case of multiple disk failures, at least one copy is available.
# Configuring
---
**For example, suppose you are interested in only one parity level of protection, and your disks are located at:**
/mnt/diskp <- selected disk for parity
/mnt/disk1 <- first disk to protect
/mnt/disk2 <- second disk to protect
/mnt/disk3 <- third disk to protect

**You must create the configuration file `/etc/snapraid.conf` with the following options:**
```bash
# --- Parity Location --- 
parity /mnt/hdd/WD-B00WR8MD/snapraid.parity 

# --- Content files ---
# One on boot drive, others on data drives
content /var/snapraid/snapraid.content
content /mnt/hdd/WD-B00WHLZD/snapraid.content
content /mnt/hdd/WD-B00WUHXD/snapraid.content

# --- Data Disks ---
data d1 /mnt/hdd/WD-B00WHLZD
data d2 /mnt/hdd/WD-B00WUHXD

# --- Excludes ---
exclude *.unrecoverable
exclude /tmp/
exclude /lost+found/
```

# Usage 
---
### Syncing 
**run the `sync` command to build the parity information.**
```bash
snapraid sync
```

>[!TIP] This process may take several hours the first time, depending on the size of the data already present on the disks. If the disks are empty, the process is immediate.
>
- You can stop it at any time by pressing `Ctrl+C`, and at the next run, it will resume where it was interrupted.

### Scrubbing
**To periodically check the data and parity for errors, you can run the `scrub` command.**
```bash 
snapraid scrub
```

- Each run of the command **checks approximately 8% of the array**, excluding data already scrubbed in the previous 10 days. You can use the `-p, --plan option` to specify a different amount and the `-o, --older-than` option to specify a different age in days. For example, to check 5% of the array for blocks older than 20 days, use:
```bash
snapraid -p 5 -o 20 scrub
  ```
- If **silent or input/output errors** are found during the process, the corresponding blocks are marked as bad in the "content" file and listed in the `status` command.
```bash
  snapraid status
  ```

- To **fix** them, you can use the `fix` command, filtering for bad blocks with the -e, --filter-error option:
```bash 
snapraid -e fix
```

- At the next "**scrub**," the errors will disappear from the "**status**" report if they are truly fixed. To make it faster, you can use `-p bad` to scrub only blocks marked as bad.
```bash 
snapraid -p bad scrub
```

Running `scrub` on an unsynced array may report errors **caused by removed or modified files**. These errors are **reported** in the `scrub` output, **but the related blocks are not marked as bad**.

### Undeleting
- SnapRAID functions more like a backup program than a RAID system, and it can be used to **restore or undelete** files to their previous state using the `-f, --filter` option:
```bash 
snapraid fix -f FILE

# OR FOR A DIRECTORY 

snapraid fix -f DIR/
```
- You can also use it to recover only accidentally deleted files inside a directory using the `-m, --filter-missing` option, which **restores only missing files, leaving all others untouched.**
```bash
snapraid fix -m -f DIR/
```
- Or to **recover all the deleted files on all drives** with:
```bash
snapraid fix -m
```

# Recovering
>[!IMPORTANT] DO NOT PANIC! You will be able to recover them!

- **The first thing you must do is avoid further changes to your disk array. Disable any remote connections to it and any scheduled processes, including any scheduled SnapRAID nightly sync or scrub.**

### STEP 1 -> Reconfigure
>[!NOTE] You need some space to recover, ideally on additional spare disks, but an external USB disk or remote disk will suffice.

- **Modify the SnapRAID configuration file** to make the "**data**" or "**parity**" option of the failed disk point to a location with enough empty space to recover the files.

>For example, if disk "d1" has failed, change from:
>`data d1 /mnt/disk1/`
>to:
>`data d1 /mnt/new_spare_disk/`

If the disk to recover is a parity disk, update the appropriate "parity" option. If you have multiple failed disks, update all their configuration options.

### Step 2 -> Fix
- Run the `fix` command, storing the log in an external file with:
```bash
snapraid -d NAME -l fix.log fix
```

Where **NAME** is the name of the disk, such as "`d1`" in our previous example. If the disk to recover is a **parity** disk, use the names "`parity`", "`2-parity`", etc. If you have multiple failed disks, use multiple `-d` options to specify all of them.

**This command will take a long time.**

Ensure you have a few gigabytes free to store the `fix.log` file. Run it from a disk with sufficient free space.

Now you have recovered all that is recoverable. If some files are **partially or totally unrecoverable**, they will be renamed by adding the ".unrecoverable" extension.

You can find a **detailed list of all unrecoverable blocks in the fix.log** file by checking all lines starting with "unrecoverable:".

If you are not satisfied with the recovery, you can retry it as many times as you wish.

For example, if you have removed files from the array after the last "sync", this may result in some files not being recovered. In this case, you can retry the "`fix`" using the `-i, --import` option, specifying where these files are now to include them again in the recovery process.

If you are satisfied with the recovery, you can proceed further, but **note that after syncing, you cannot retry the "fix" command anymore!**

### Step 3 -> Check 
- As a cautious check, you can now run a "check" command to ensure that everything is correct on the recovered disk.
```bash
snapraid -d NAME -a check
```
Where `NAME` is the name of the disk, such as "`d1`" in our previous example.

The `-d` and `-a` options tell SnapRAID to check only the specified disk and ignore all parity data.

**This command will take a long time**, but if you are not overly cautious, you can skip it.

### Step 4 -> Sync
Run the "`sync`" command to resynchronize the array with the new disk.

```bash
snapraid sync
```

# Commands
- For further information on commands refer to
  https://www.snapraid.it/manual






