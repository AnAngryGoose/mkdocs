# DIY NAS: MergerFS + SnapRAID

* **Sources:**
* [SnapRAID Manual](https://www.snapraid.it/manual)
* [MergerFS Documentation](https://github.com/trapexit/mergerfs)
* [Ubuntu Server Storage](https://www.google.com/search?q=https://ubuntu.com/server/docs/storage-introduction)

---
## You can find more in depth info in the related guides here. 
* This is just how I did it. There are plenty of other ways to do it using different configurations (ZFS, Proxmox, Unraid, TrueNAS, OMV, etc). All options have pros and cons. 

* For my configuration I just created an ubuntu server with MergerFS + SnapRAID. 

    **MergerFS:** Pool drives and read them as one drive. FUSE system. Various configurations, read the official docs. 

    **SnapRAID:** Create routine snapshots of drives to restore them in case of failure. No striping, easy to expand, straightforward to manage. 


### 1. Drive Preparation

Before pooling, drives must be partitioned and formatted.

* **Note:** Identify your disks carefully using `lsblk` or `sudo fdisk -l`.
* **Strategy:** Use **EXT4** for simplicity and reliability on Ubuntu.

**Format a Drive (Repeat for each new disk):**

```bash
# 1. Create a GPT partition table (Destructive!)
sudo parted /dev/sdb mklabel gpt

# 2. Create the partition (100% of disk)
sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%

# 3. Format to EXT4
# -m 0: Reserve 0% for root (max space for data)
# -L: Label the disk (disk1, disk2, parity1) to make it easy to identify
sudo mkfs.ext4 -m 0 -L disk1 /dev/sdb1

```

*(Repeat for `disk2`, `parity1`, etc.)*

---

### 2. Permanent Mounting (`/etc/fstab`)

Do not mount by `/dev/sdb1` as these names change on reboot. Use `UUID` or `LABEL`.

**A. Create Mount Points**

```bash
sudo mkdir -p /mnt/disk1 /mnt/disk2 /mnt/parity1

```

**B. Edit Fstab**
Open the config: `sudo nano /etc/fstab`
Add lines for your specific drives:

```bash
# Data Disks
LABEL=disk1    /mnt/disk1    ext4    defaults,nofail    0 2
LABEL=disk2    /mnt/disk2    ext4    defaults,nofail    0 2

# Parity Disk (Must be largest drive)
LABEL=parity1  /mnt/parity1  ext4    defaults,nofail    0 2

```

* **nofail:** Critical for headless servers. If a drive dies, the server will still boot.
* **Test:** Run `sudo mount -a` to verify no errors.

---

### 3. MergerFS (The Pool)

MergerFS combines multiple data drives into one virtual folder. It does **not** strip data; if one drive fails, you only lose files on *that* drive.

**A. Install MergerFS**

```bash
sudo apt update && sudo apt install mergerfs -y

```

**B. Configure Pool in Fstab**
Add this line to the bottom of `/etc/fstab`:

```bash
# Syntax: /mnt/disk* (all mounts starting with disk) -> /mnt/storage
/mnt/disk* /mnt/storage  fuse.mergerfs  defaults,nonempty,allow_other,use_ino,cache.files=partial,moveonenospc=true,dropcacheonclose=true,minfreespace=50G,fsname=mergerfs  0 0

```

* **minfreespace=50G:** Prevents filling a drive completely, which creates fragmentation issues.
* **allow_other:** Allows software (Plex/Samba) to access the pool.

**C. Activate**

```bash
sudo mkdir -p /mnt/storage
sudo mount -a

```

---

### 4. SnapRAID (The Protection)

SnapRAID calculates parity (like RAID 5) but does it on a schedule (Snapshot), not in real-time. Ideally suited for media servers where files rarely change.

**A. Install SnapRAID**
Ubuntu repos often have outdated versions. It is safer to build from source or download the latest .deb.

```bash
# Check version in repo or download from official site
sudo apt install snapraid -y
snapraid --version

```

**B. Configure `/etc/snapraid.conf**`
Backup the default and create your own: `sudo nano /etc/snapraid.conf`

```text
# 1. Parity Location (The dedicated large drive)
parity /mnt/parity1/snapraid.parity

# 2. Content Files (Index of all your files)
# Save multiple copies on different physical disks!
content /var/snapraid.content
content /mnt/disk1/.snapraid.content
content /mnt/disk2/.snapraid.content

# 3. Data Disks ( The drives you want to protect)
data d1 /mnt/disk1/
data d2 /mnt/disk2/

# 4. Excludes (Temp files/Trash)
exclude *.unrecoverable
exclude /tmp/
exclude /lost+found/

```

**C. Initialize**
Run the first sync to calculate parity. This will take hours depending on data size.

```bash
sudo snapraid sync

```

---

### 5. Automation & Maintenance

SnapRAID is manual by default. You must automate it to stay protected.

**A. Basic Crontab**
Edit cron: `sudo crontab -e`
Add a nightly sync and weekly scrub (checks for bit-rot).

```bash
# Sync every night at 3 AM
0 3 * * * snapraid sync >> /var/log/snapraid.log

# Scrub (check data integrity) every Sunday at 5 AM
0 5 * * 0 snapraid scrub -p 5 >> /var/log/snapraid_scrub.log

```

* **-p 5:** Scrubs 5% of the array each week, ensuring a full check every ~5 months.

---

### Troubleshooting

* **"Disk is full" during sync:** Your parity drive must be equal to or larger than your single largest data drive.
* **MergerFS permissions:** If Docker containers cannot see files, ensure `allow_other` is in fstab and check PUID/PGID matches the folder owner.
* **Data Recovery:** If `disk1` fails, replace it, mount to `/mnt/disk1`, and run `snapraid fix -d d1`.

[Comprehensive SnapRAID & MergerFS Guide](https://www.youtube.com/watch?v=n7piuhTXeG4)

This is a solid video on getting MergerFS running well. 