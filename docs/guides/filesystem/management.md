# Filesystem Management Guide

---
tags:
  - filesystem
  - fstab
  - storage
  - nas
  - infrastructure
---


# Identify Drives
---
```bash
lsblk -o NAME,MODEL,SIZE,TYPE,FSTYPE
```

**Output should read like:**
```bash
NAME                      MODEL                            SIZE TYPE FSTYPE
sda                       SanDisk SD8TB8U2               238.5G disk 
└─sda1                                                   238.5G part ext4
sdb                       WDC WD120EFGX-68                10.9T disk 
└─sdb1                                                    10.9T part xfs
sdc                       WDC WD120EFGX-68                10.9T disk 
└─sdc1                                                    10.9T part xfs
sdd                       WDC WD120EFGX-68                10.9T disk 
└─sdd1                                                    10.9T part xfs
nvme0n1                   WDC PC SN730 SDBQNTY-256G-1001 238.5G disk 
├─nvme0n1p1                                                  1M part 
├─nvme0n1p2                                                  2G part ext4
└─nvme0n1p3                                              236.5G part LVM2_member
  └─ubuntu--vg-ubuntu--lv                                  100G lvm  ext4
```

# Formatting & Partitioning Drives
---
- ### **Formatting SSD as ext4**
```bash
# Wipe partition table
sudo wipefs -a /dev/sda

# Create new partition table and partition
echo -e "n\np\n1\n\n\nw" | sudo fdisk /dev/sda
# OR

# create partition table 
sudo parted -s /dev/sda mklabel gpt 



# Format to EXT4
sudo mkfs.ext4 -L "appdata" /dev/sda1
```

##### **NOTE on fdisk params above:**

`fdisk` is a tool that usually asks you questions one by one (like "Do you want a new partition?" or "Start sector?"). This command pre-writes all your answers and pipes them into the tool so it runs instantly without waiting for you to type.

- **`echo -e`**: "Print this text, and treat `\n` as the Enter key."
- **`g`**: Create a new **GPT** partition table (Solves the >2TB error).
- **`n`**: Select **N**ew partition.
- **`\n`**: Press **Enter**.
- **`p`**: Select **P**rimary partition type.
- **`\n`**: Press **Enter**.
- **`1`**: Select Partition Number **1**.
- **`\n`**: Press **Enter**.
- **`\n`**: Press **Enter** (Accept default _First Sector_ / Start of disk).
- **`\n`**: Press **Enter** (Accept default _Last Sector_ / End of disk).
- **`w`**: **W**rite changes to disk and exit.

- ### **Formatting 3x HDDs as XFS** 
```bash
# Drive 1 (sdb)
# wipe fs
sudo wipefs -a /dev/sdb
# create partition
sudo parted -s /dev/sdb mkpart primary xfs 0% 100%
# list partition
sudo fdisk -l /dev/sdb
# format partition
sudo mkfs.xfs -f -L "disk1" /dev/sdb1

# Drive 2 (sdc)
sudo wipefs -a /dev/sdc
sudo parted -s /dev/sdc mkpart primary xfs 0% 100%
sudo fdisk -l /dev/sdc
sudo mkfs.xfs -f -L "disk2" /dev/sdc1

# Drive 3 (sdd)
sudo wipefs -a /dev/sdd
sudo parted -s /dev/sdd mkpart primary xfs 0% 100%
sudo fdisk -l /dev/sdd  
sudo mkfs.xfs -f -L "parity1" /dev/sdd1
```

# Get UUIDs
```bash
ls -l /dev/disk/by-uuid/
```
- **Output will be:**
```bash 
lrwxrwxrwx 1 root root 10 Dec  7 05:34 02de315f-8ced-4745-bb4f-2d24c46efa48 -> ../../sdb1
lrwxrwxrwx 1 root root 15 Dec  7 04:33 3c26dce0-7f10-426c-bd67-e9f2d77889ee -> ../../nvme0n1p2
lrwxrwxrwx 1 root root 10 Dec  7 05:34 603999da-c2dd-484d-80bf-ab4977680e90 -> ../../sdc1
lrwxrwxrwx 1 root root 10 Dec  7 05:34 844cae61-9599-4a57-9431-18c0a33904e4 -> ../../sda1
lrwxrwxrwx 1 root root 10 Dec  7 05:34 9860056e-b314-4cb9-acc5-30e76281795b -> ../../sdd1
lrwxrwxrwx 1 root root 10 Dec  7 04:33 d5ac7faa-876c-43ff-b27a-94c53bf79bb2 -> ../../dm-0
```
# Mounting
- **Create Mount points:**
```bash
# Must make the folder to mount drives to
sudo mkdir -p /mnt/disk1
sudo mkdir -p /mnt/disk2
sudo mkdir -p /mnt/parity1
sudo mkdir -p /mnt/appdata

# Mount Parition
sudo mount /dev/sdb1 /mnt/parity1
sudo mount /dev/sdc1 /mnt/disk1
sudo mount /dev/sdd1 /mnt/disk2

# Verify Mounting 
df -h 
# Filesystem      Size  Used Avail Use% Mounted on
# ...
# /dev/sda1       7.3T   93M  6.9T   1% /mnt/mydrive

```

- **Edit fstab**
	- Scroll to bottom of fstab and add: 
```bash
# ---------------------------------------------------------
# DATA DRIVES (XFS) - 12TB WD Reds
# ---------------------------------------------------------
# Disk 1 (sdb1)
UUID=02de315f-8ced-4745-bb4f-2d24c46efa48 /mnt/disk1 xfs defaults,noatime 0 0

# Disk 2 (sdc1)
UUID=603999da-c2dd-484d-80bf-ab4977680e90 /mnt/disk2 xfs defaults,noatime 0 0

# Parity Drive (sdd1)
UUID=9860056e-b314-4cb9-acc5-30e76281795b /mnt/parity1 xfs defaults,noatime 0 0

# ---------------------------------------------------------
# FAST STORAGE (EXT4) - 240GB SSD
# ---------------------------------------------------------
# Appdata/Cache (sda1)
UUID=844cae61-9599-4a57-9431-18c0a33904e4 /mnt/appdata ext4 defaults,noatime 0 0
```
- ##### ***NOTE:  MUST RUN AFTER EDITING fstab***
```bash
# verify syntax 
sudo findmnt --verify

# reload fstab
sudo systemctl daemon-reload
```
# Verify 
- **Test this BEFORE reboot** - no output is good
```bash
sudo mount -a
```

- **Check correct sizing:**
```bash
df -h
```

 You should see:

- `disk1`, `disk2`, `parity1`: ~11T
    
- `appdata`: ~220G
  
- **Check ownership:**
```bash
ls -ld /mnt/disk1 /mnt/disk2 /mnt/parity1 /mnt/appdata
```

- If owner listed as `root` (and doesnt need to be) run :
```bash 
sudo chown -R goose:goose /mnt/disk1 /mnt/disk2 /mnt/parity1 /mnt/appdata
```
### OPTIONAL: cfdisk - interactive formatting
**1. For the SSD (`/dev/sda`):**

```Bash
sudo cfdisk /dev/sda
```

- **Select Label Type:** Choose **`gpt`** (it may auto-select).
- **Delete:** If you see existing partitions, highlight them and select **[ Delete ]** at the bottom.
- **New:** Select **[ New ]** -> **[ Enter ]** (Use max size).
- **Write:** Select **[ Write ]** -> Type `yes` to confirm.
- **Quit:** Select **[ Quit ]**.
    
**2. For the HDDs (/dev/sdb, etc.):**

```Bash
sudo cfdisk /dev/sdb
sudo cfdisk /dev/sdc
sudo cfdisk /dev/sdd
```

_(Always choose `gpt` if asked)._

---
#### Don't forget to Format!

`cfdisk` only creates the _partition_ (the container). It does **not** create the filesystem (the XFS/EXT4 formatting).

Once you finish with `cfdisk` for all drives, you **must** still run the format commands I listed earlier:

Bash

```
# Format the SSD
sudo mkfs.ext4 -L "appdata" /dev/sda1

# Format the HDDs
sudo mkfs.xfs -f -L "disk1" /dev/sdb1
sudo mkfs.xfs -f -L "disk2" /dev/sdc1
sudo mkfs.xfs -f -L "parity1" /dev/sdd1
```


# General CLI Management & Cheat Sheet

### Identification & Hardware Info

| **Command**             | **Description**                                                                                       |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| `lsblk`                 | **List Block Devices.** The best "at a glance" view of disks, partitions, and where they are mounted. |
| `lsblk -f`              | Lists devices plus **filesystem types** (ext4, ntfs) and **UUIDs**.                                   |
| `sudo fdisk -l`         | Detailed low-level partition table dump. Good for verifying sector sizes.                             |
| `sudo lshw -class disk` | Hardware details. Shows model numbers, serial numbers, and firmware versions.                         |
| `sudo blkid`            | Prints the UUIDs and labels for all block devices.                                                    |
### Disk Usage & Space

|**Command**|**Description**|
|---|---|
|`df -h`|**Disk Free.** Shows total size, used space, and available space for all mounted drives in "Human Readable" format (GB/TB).|
|`du -sh /path/to/dir`|**Disk Usage.** Calculates the size of a specific directory. Useful for finding what is eating your space.|
|`ncdu`|**Interactive Usage.** (Requires install). A visual, navigable tool to find large files. Highly recommended.|

### Health & Maintenance

|**Command**|**Description**|
|---|---|
|`sudo smartctl -a /dev/sda`|**SMART Data.** (Requires `smartmontools`). Checks the physical health, temperature, and error logs of the drive.|
|`sudo fsck /dev/sda1`|**File System Check.** Like Windows chkdsk. **Only run on unmounted drives.** Repairs corruption.|

### Mounting Operations

|**Command**|**Description**|
|---|---|
|`sudo mount /dev/sdb1 /mnt/usb`|Manually mounts a partition to a folder.|
|`sudo umount /mnt/usb`|Unmounts the drive safely (e.g., before unplugging).|
|`sudo mount -a`|Mounts everything listed in `/etc/fstab` that isn't already mounted.|

---

## 3. Docker Integration Reference

Once a drive is mounted on the host (e.g., `/mnt/nvme_storage`), you must explicitly pass it to containers that need it.

### For FileBrowser (To manage files)

YAML

```
services:
  filebrowser:
    volumes:
      # Map Host Path : Container Path
      - /mnt/nvme_storage:/mnt/nvme_storage
```

### For Beszel (To monitor disk I/O and space)

YAML

```
services:
  beszel-agent:
    volumes:
      - /mnt/nvme_storage:/mnt/nvme_storage:ro
    environment:
      # Comma-separated list of mount points to monitor
      - EXTRA_FILESYSTEMS=/mnt/nvme_storage
```

---