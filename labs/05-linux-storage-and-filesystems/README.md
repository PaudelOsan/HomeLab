# Lab 05 — Linux Storage & Filesystems

## Overview

This lab covers Linux storage devices, partitions, filesystems, filesystem UUIDs, mount points, persistent mount configuration, disk health, and safe storage inspection.

The practical exercises were performed on a Lenovo ThinkCentre M800 SFF running Ubuntu 26.04.1 LTS.

### Learning objectives

By completing this lab, I learned how to:

- Identify physical disks, partitions, and virtual block devices.
- Understand the relationship between disks, partitions, filesystems, and mount points.
- Check filesystem capacity and available space.
- Identify filesystem types and UUIDs.
- Inspect partition tables without modifying disks.
- Check HDD health using SMART diagnostics.
- Understand the purpose of `/etc/fstab`.
- Compare configured mounts with currently active mounts.
- Recognize storage-related risks before formatting or modifying a disk.
- Interpret common Linux storage commands and their output.

---

# 1. Storage fundamentals

## 1.1 Physical disk

A physical disk is a storage device such as an HDD, SATA SSD, or NVMe SSD.

Examples from my lab:

- `/dev/nvme0n1` — NVMe SSD containing Ubuntu.
- `/dev/sdb` — approximately 179 GB SATA SSD containing Windows-related partitions.
- `/dev/sda` — approximately 1 TB Seagate HDD.

Linux exposes storage devices through device files, usually under `/dev/`.

A device name identifies a device, but it does not by itself tell me whether the device contains important data.

## 1.2 Partition

A partition is a defined region of a disk.

For example:

- `/dev/nvme0n1p1` is partition 1 on the NVMe drive.
- `/dev/nvme0n1p2` is partition 2 on the NVMe drive.
- `/dev/sdb3` is partition 3 on the SATA drive `/dev/sdb`.

The naming convention differs between devices:

- SATA-style disk: `/dev/sda`, with partitions such as `/dev/sda1`.
- NVMe disk: `/dev/nvme0n1`, with partitions such as `/dev/nvme0n1p1`.

A disk can exist without any partitions being listed.

## 1.3 Partition table

A partition table records the layout of partitions on a disk.

Common partition table types include:

- **GPT:** GUID Partition Table, commonly used on modern computers with UEFI firmware.
- **MBR:** Master Boot Record partitioning scheme, an older format.

Finding GPT metadata does not prove that a disk has partitions or that it contains no recoverable data.

## 1.4 Filesystem

A filesystem organizes data into files and directories.

Filesystems encountered in this lab:

| Filesystem | General purpose |
|---|---|
| `ext4` | Common Linux filesystem |
| `vfat` | Filesystem format commonly used for EFI system partitions |
| `ntfs` | Common Windows filesystem |
| `tmpfs` | Temporary filesystem backed primarily by memory |
| `swap` | Space used by Linux for memory-management purposes |

A partition does not necessarily have a recognized filesystem. A disk can also contain filesystem or partition information that is not currently mounted.

## 1.5 Mount point

Linux makes a filesystem accessible by attaching it to a directory called a mount point.

Examples:

- `/` — the root filesystem of Ubuntu.
- `/boot/efi` — the EFI system partition mount point.
- `/mnt/storage` — an example of a directory that could be used as a mount point for additional storage.

Unlike Windows, Linux does not normally assign a separate drive letter to every filesystem. Multiple filesystems can be attached to different directories in one directory tree.

## 1.6 Understanding the complete relationship

A typical storage arrangement is:

**Physical disk → Partition → Filesystem → Mount point**

For my Ubuntu installation:

`/dev/nvme0n1` → `/dev/nvme0n1p2` → `ext4` → `/`

This is a conceptual relationship. Some storage arrangements, such as filesystems directly on whole disks, logical volumes, RAID devices, and network filesystems, can differ.

---

# 2. My homelab storage layout

The following table summarizes the storage devices discovered during the lab.

| Device | Approximate size | Observed configuration |
|---|---:|---|
| `/dev/nvme0n1` | 238.5 GiB | Ubuntu system NVMe |
| `/dev/nvme0n1p1` | 1 GiB | EFI partition, `vfat` |
| `/dev/nvme0n1p2` | 237.4 GiB | Ubuntu root filesystem, `ext4` |
| `/dev/sdb` | 178.9 GiB | SATA SSD with Windows-related partitions |
| `/dev/sdb1` | 200 MiB | `vfat` partition |
| `/dev/sdb2` | 16 MiB | Microsoft reserved partition; no filesystem type reported |
| `/dev/sdb3` | 177.8 GiB | `ntfs` partition |
| `/dev/sdb4` | 892 MiB | `ntfs` partition |
| `/dev/sda` | 931.5 GiB | Seagate HDD with GPT metadata but no listed partitions |

Sizes may differ slightly between manufacturers' decimal units and the binary units displayed by Linux.

## Important observations

- Ubuntu runs from `/dev/nvme0n1p2`.
- The EFI partition is mounted at `/boot/efi`.
- The Windows-related SATA SSD partitions were not mounted during the checks.
- `/dev/sda` had no partition entries in the partition-table inspection.
- `/dev/sda` was not mounted and had no filesystem UUID or filesystem type reported by `blkid`.
- The existence of a partition table does not prove the presence of partitions.
- The absence of listed partitions does not prove that the disk is empty or that previous data is unrecoverable.

---

# 3. Command reference — identifying disks and filesystems

This section is designed as a future reference. Each command includes its purpose, syntax, important flags, and interpretation.

## 3.1 `lsblk` — list block devices

### Command

```bash
lsblk
```

### What it does

`lsblk` lists block devices such as HDDs, SSDs, partitions, and loop devices. Its tree structure helps show which partitions belong to which disks.

### Useful options

| Option | Meaning |
|---|---|
| `-f` | Show filesystem-related information, including filesystem type, label, UUID, and mount points |
| `-l` | Use list format instead of the normal tree layout |
| `-o` | Select the columns to display |
| `-p` | Display full device paths, such as `/dev/sda` |
| `-a` | Include empty devices as well as devices that would normally be omitted |

### Example

```bash
lsblk -f
```

The `-f` option was particularly useful because it exposed filesystem types and UUIDs alongside the storage layout.

To display only selected columns:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

This requests the device name, size, device type, filesystem type, and mount points.

### How to interpret the output

| Column | Meaning |
|---|---|
| `NAME` | Device or partition name |
| `SIZE` | Reported capacity |
| `TYPE` | Device type, such as `disk`, `part`, or `loop` |
| `FSTYPE` | Recognized filesystem type |
| `UUID` | Filesystem UUID, if available |
| `MOUNTPOINTS` | Directories where the filesystem is mounted |

A `disk` entry represents a disk device. A `part` entry represents a partition. A `loop` entry represents a virtual block device backed by another file.

My Ubuntu root partition appeared as `/dev/nvme0n1p2`, with filesystem type `ext4` and mount point `/`.

### Important note about loop devices

My output contained several `loop` devices associated with Snap packages. These are normal in Ubuntu and do not indicate that additional physical disks are installed.

---

## 3.2 `df -h` — filesystem capacity and usage

### Command

```bash
df -h
```

### What it does

`df` reports disk space usage for mounted filesystems.

The `-h` option means **human-readable**. It displays sizes in convenient units such as K, M, G, or T rather than showing only large numbers of bytes.

### Important columns

| Column | Meaning |
|---|---|
| `Filesystem` | Filesystem or device providing the storage |
| `Size` | Reported filesystem capacity |
| `Used` | Space currently used |
| `Avail` | Space available to ordinary users |
| `Use%` | Percentage of capacity used |
| `Mounted on` | Mount point |

### My actual findings

The root filesystem showed:

```text
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/nvme0n1p2  233G   16G   205G    8%  /
```

Interpretation:

- The Ubuntu root filesystem had approximately 233G of reported capacity.
- Approximately 16G was used.
- Approximately 205G was available.
- The reported usage was 8%.

These values describe the mounted filesystem, not necessarily the full physical disk capacity.

### What about `tmpfs`?

My output also included entries such as `/run`, `/dev/shm`, and `/tmp`.

`tmpfs` is a temporary filesystem backed primarily by memory. It should not be confused with an additional physical HDD or SSD.

### When to use `df -h`

Use it when investigating:

- A server running out of storage.
- Applications failing because a filesystem is full.
- Log files consuming excessive space.
- Whether a mounted filesystem has enough capacity for a planned workload.

Remember that `df -h` reports mounted filesystems; it is not a complete inventory of every physical disk.

---

## 3.3 `lsblk -f` — identify filesystem types and UUIDs

### Command

```bash
lsblk -f
```

This combines the block-device layout with filesystem information.

My output identified:

- `/dev/nvme0n1p1` as `vfat`, mounted at `/boot/efi`.
- `/dev/nvme0n1p2` as `ext4`, mounted at `/`.
- `/dev/sdb3` and `/dev/sdb4` as `ntfs`.
- `/dev/sda` without a recognized filesystem type.

### Why this matters

A device's name alone does not tell me what filesystem it contains or whether it is mounted.

`lsblk -f` helps answer:

- Which partitions have recognized filesystems?
- What type of filesystem is present?
- Which filesystems are mounted?
- Which UUID belongs to each filesystem?

It is one of my first commands to use when investigating unfamiliar storage.

---

# 4. Command reference — filesystem identification and partition tables

## 4.1 `sudo blkid` — identify filesystems

### Command

```bash
sudo blkid
```

### What it does

`blkid` displays block-device identification information, including recognized filesystem types, UUIDs, and partition identifiers.

The `sudo` prefix runs the command with elevated privileges. Depending on system permissions and available cached information, this can allow more complete device inspection.

### Understanding the important fields

Example from my system:

```text
/dev/nvme0n1p2: UUID="caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="c046cef4-410a-4fe0-a0c0-9c9c120d94f7"
```

| Field | Meaning |
|---|---|
| `UUID` | Identifier for the filesystem |
| `BLOCK_SIZE` | Filesystem block-size information reported by the tool |
| `TYPE` | Recognized filesystem type |
| `PARTUUID` | Identifier for the partition, distinct from the filesystem UUID |

### UUID versus PARTUUID

These identifiers serve different purposes.

- **Filesystem UUID:** Identifies the filesystem.
- **PARTUUID:** Identifies the partition through its partition-table metadata.

The filesystem UUID is commonly used in `/etc/fstab` so Linux can refer to a filesystem without depending solely on a device name.

A filesystem UUID and a PARTUUID should not be treated as interchangeable.

### My root filesystem UUID

```text
caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65
```

The same UUID appeared in `/etc/fstab`, confirming that the configured root entry referred to the filesystem identified by `blkid`.

### What did `/dev/sda` show?

The output included:

```text
/dev/sda: PTUUID="e90fa2b5-9f2f-4715-a8dc-e098738f910c" PTTYPE="gpt"
```

This reported a GPT partition-table identifier and table type. It did not report a recognized filesystem `TYPE` or filesystem `UUID`.

This is consistent with the other findings, but it does not establish whether the disk has recoverable data.

---

## 4.2 `sudo fdisk -l` — inspect partition tables

### Command used

```bash
sudo fdisk -l /dev/sda
```

### What it does

`fdisk` is a disk-partitioning utility. Its `-l` option lists partition-table information rather than entering the interactive partition-editing interface.

Here, `/dev/sda` specifies the particular disk being inspected.

### Important options and arguments

| Part | Meaning |
|---|---|
| `sudo` | Run with elevated privileges |
| `fdisk` | Partition-table utility |
| `-l` | List disk and partition information |
| `/dev/sda` | Target disk |

### My findings

The output reported:

- Disk model: `ST1000DM003-1ER1`
- Capacity: approximately 931.51 GiB
- Sector size: 512 bytes logical and 4096 bytes physical
- Disklabel type: `gpt`
- No partition entries listed below the partition-table headings

### Interpretation

The disk was detected, and GPT partition-table metadata was present. However, `fdisk -l` did not list any defined partitions.

This was an inspection-only command in this exercise. I did not create, delete, or change any partitions.

---

## 4.3 `sudo sgdisk -p /dev/sda` — print GPT information

### Command used

```bash
sudo sgdisk -p /dev/sda
```

### What it does

`sgdisk` is a utility for inspecting and managing GPT partition tables.

The `-p` option prints the partition-table layout.

### My findings

The output reported:

- Disk model: `ST1000DM003-1ER1`
- Capacity: approximately 931.5 GiB
- Partition-table identifier: `E90FA2B5-9F2F-4715-A8DC-E098738F910C`
- Total free space reported within the usable GPT partition area: approximately 931.5 GiB
- No partition rows

### Interpretation

The partition table contained no listed partitions, and the utility reported the disk's usable partition area as free.

**This does not prove that the disk contains no recoverable data.** Old filesystem structures, previous partition layouts, or other data may still exist on unallocated areas of a disk.

### Safety lesson

Inspection commands should come before any attempt to initialize, partition, or format an unfamiliar drive.

I did not use `sgdisk` to modify the partition table.

---

# 5. SMART disk health diagnostics

## 5.1 What is SMART?

SMART stands for **Self-Monitoring, Analysis and Reporting Technology**.

It allows compatible storage devices to report health-related attributes, error logs, and self-test results.

SMART is useful for identifying warning signs, but it cannot guarantee that a drive will remain reliable.

## 5.2 Check whether `smartmontools` is installed

### Command

```bash
dpkg-query -W smartmontools
```

`dpkg-query` queries the package database.

- `-W` requests package information.
- `smartmontools` is the package name.

Initially, the package was not installed, and the query reported that the package was not installed and no package information was available.

I then checked package availability with:

```bash
apt-cache policy smartmontools
```

`apt-cache` queries information available in the package manager's local metadata.

- `policy` displays package version and repository-priority information.
- `smartmontools` specifies the package being checked.

The package candidate shown was version `7.5-2`.

I installed it using:

```bash
sudo apt install smartmontools
```

Explanation:

- `sudo` runs the package-management operation with elevated privileges.
- `apt` is Ubuntu's package-management command-line tool.
- `install` requests installation of a package.
- `smartmontools` is the package to install.

Installing a diagnostic tool changes the system's software state, but it does not format or repartition the inspected HDD.

## 5.3 Inspect the HDD's SMART report

### Command

```bash
sudo smartctl -a /dev/sda
```

### What it does

`smartctl` reads and displays SMART information for a compatible storage device.

- `-a` requests all SMART information supported by the utility for the device, including available attributes and logs.
- `/dev/sda` identifies the target HDD.

The `-a` option is an information-display option, not an instruction to erase or repair the drive.

### My actual findings

The report identified a Seagate Barracuda 7200.14 drive, model `ST1000DM003-1ER162`, with firmware `CC46`.

The report included:

- SMART support available and enabled.
- Overall SMART health self-assessment: `PASSED`.
- A recorded short offline test completed without error.
- Reallocated sector count: raw value `0`.
- Current pending sector count: raw value `0`.
- Offline uncorrectable sector count: raw value `0`.
- UDMA CRC error count: raw value `0`.
- Power-on hours: approximately `54,459`.
- Temperature: approximately `28°C`.
- Reported uncorrectable errors: raw value `77`.
- ATA error log count: `77`, with historical errors displayed.

The error log contained repeated historical errors, including uncorrectable-read (`UNC`) entries.

## 5.4 Understanding SMART output

### Overall health

`PASSED` means the drive has not reported a failure according to its overall SMART health assessment.

It does **not** mean the drive is guaranteed healthy or safe for important data.

### Important attributes

| Attribute | Meaning | Why it matters |
|---|---|---|
| `Reallocated_Sector_Ct` | Sectors replaced after being found unreliable | A nonzero or increasing count can indicate media problems |
| `Current_Pending_Sector` | Sectors awaiting a decision about whether they can be read reliably or remapped | Pending sectors may indicate read problems |
| `Offline_Uncorrectable` | Uncorrectable errors found during offline operations | Can indicate problematic sectors |
| `UDMA_CRC_Error_Count` | Interface data-transfer errors detected using CRC | Can help investigate cables or connections |
| `Power_On_Hours` | Reported accumulated powered-on time | Helps estimate drive usage |
| `Reported_Uncorrect` | Errors reported to the host that could not be corrected | Historical or increasing errors deserve attention |
| `Command_Timeout` | Commands that did not complete within the expected time | Can have several causes and requires context |

### Important caution about raw values

Some SMART attributes use vendor-specific raw-value formats. Seagate's raw read-error and seek-error values should not automatically be interpreted as literal counts of failed operations.

Normalized values, thresholds, raw values, and historical logs must be interpreted together. A single attribute is not enough to establish a drive's complete health.

### My assessment of `/dev/sda`

Although the overall SMART assessment was `PASSED`, the historical uncorrectable errors and error log are concerning.

My conclusion is to **avoid relying on this HDD for important storage, backups, or future NAS data**.

I did not perform a repair or destructive test. The drive was inspected only.

If the disk may contain data that matters, it should not be formatted or subjected to unnecessary write operations. Recovery considerations should come before reuse.

---

# 6. Mounts and filesystem configuration

## 6.1 What is mounting?

Mounting attaches a filesystem to a directory in the Linux directory tree.

For example, mounting a filesystem at `/mnt/storage` would make its files accessible through that directory.

A mount point is a directory, not a disk itself.

## 6.2 `findmnt` — inspect mounted filesystems

### General command

```bash
findmnt
```

This lists currently mounted filesystems and their relationships.

My output showed `/` coming from `/dev/nvme0n1p2` with filesystem type `ext4`.

### Inspect the root filesystem

```bash
findmnt /
```

The `/` argument requests information about the mount associated with the root directory.

### My output

```text
TARGET  SOURCE           FSTYPE  OPTIONS
/       /dev/nvme0n1p2   ext4    rw,relatime
```

### Interpretation

| Field | Meaning |
|---|---|
| `TARGET` | Mount point |
| `SOURCE` | Device or filesystem providing the mount |
| `FSTYPE` | Filesystem type |
| `OPTIONS` | Active mount options |

The output confirmed that Ubuntu's root directory was currently mounted from `/dev/nvme0n1p2` using `ext4`.

### Understanding `rw,relatime`

- `rw` means the filesystem is mounted read-write.
- `relatime` means access timestamps are updated according to a performance-conscious policy rather than necessarily being updated on every read.

This describes the active mount options reported by the system.

## 6.3 `/etc/fstab` — persistent mount configuration

### Command

```bash
cat /etc/fstab
```

### What it does

`/etc/fstab` is a configuration file containing filesystem mount entries and swap configuration.

It commonly tells Linux which filesystems should be mounted automatically during startup.

The command uses:

- `cat` to display file contents.
- `/etc/fstab` as the file path.

This command reads the configuration; it does not mount or unmount anything.

### My actual configuration

```text
# / was on /dev/nvme0n1p2 during curtin installation
/dev/disk/by-uuid/caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65 / ext4 defaults 0 1

# /boot/efi was on /dev/nvme0n1p1 during curtin installation
/dev/disk/by-uuid/2C95-3255 /boot/efi vfat defaults 0 1

/swap.img none swap sw 0 0
```

### Understanding the six fields in an `/etc/fstab` entry

Consider the root entry:

```text
/dev/disk/by-uuid/caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65 / ext4 defaults 0 1
```

| Field | Value | Meaning |
|---|---|---|
| 1 | `/dev/disk/by-uuid/...` | Filesystem source identified by UUID |
| 2 | `/` | Mount point |
| 3 | `ext4` | Filesystem type |
| 4 | `defaults` | Standard mount options |
| 5 | `0` | Excluded from the legacy `dump` backup mechanism |
| 6 | `1` | Filesystem-check pass priority; root is checked first when applicable |

The sixth field is associated with filesystem checking during boot. It does not mean that a check runs on every boot unconditionally.

### Understanding `defaults`

The `defaults` option requests standard mount behavior. Common defaults include read-write access and automatic mounting, subject to the filesystem type, system configuration, and applicable options.

### Understanding the EFI entry

```text
/dev/disk/by-uuid/2C95-3255 /boot/efi vfat defaults 0 1
```

This configures the filesystem identified by UUID `2C95-3255` to be mounted at `/boot/efi` as `vfat`.

### Understanding the swap entry

```text
/swap.img none swap sw 0 0
```

This entry configures a swap file:

- `/swap.img` is the swap source.
- `none` means it does not use a normal directory mount point.
- `swap` identifies the entry as swap space.
- `sw` specifies standard swap options.
- The final `0 0` fields exclude it from the legacy `dump` mechanism and filesystem-check pass.

Swap is used by Linux for memory management; it is not a regular filesystem mounted at a directory.

## 6.4 `findmnt --fstab` — inspect configured mounts

### Command

```bash
findmnt --fstab
```

### What it does

`findmnt` can inspect the mount table or read the filesystem configuration.

The `--fstab` option requests entries from `/etc/fstab`, allowing me to inspect the configured mounts.

### My output

```text
TARGET    SOURCE                                                 FSTYPE OPTIONS
/         /dev/disk/by-uuid/caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65 ext4   defaults
/boot/efi /dev/disk/by-uuid/2C95-3255                            vfat   defaults
none      /swap.img                                              swap   sw
```

### Why use it?

It provides a structured view of the configuration rather than displaying the raw text file.

The output confirmed that the configured root, EFI, and swap entries matched the entries in `/etc/fstab`.

## 6.5 Configured mounts versus active mounts

This is one of the most important concepts from the lab.

| Command | Main question answered |
|---|---|
| `cat /etc/fstab` | What is configured in the file? |
| `findmnt --fstab` | What mount entries are configured in fstab? |
| `findmnt /` | What filesystem is currently mounted at `/`? |
| `findmnt` | What filesystems are currently mounted? |

A filesystem can be configured for mounting without currently being mounted. Likewise, a filesystem can be mounted manually without having a permanent entry in `/etc/fstab`.

---

# 7. UUID verification exercise

The practical exercise connected filesystem identification with persistent mount configuration.

## Step 1: Identify the Ubuntu root partition

Command:

```bash
lsblk -f
```

Finding:

- Device: `/dev/nvme0n1p2`
- Filesystem: `ext4`
- Mount point: `/`

## Step 2: Read the filesystem UUID

Command:

```bash
sudo blkid
```

Finding:

```text
UUID="caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65"
TYPE="ext4"
```

## Step 3: Read the configured root mount

Command:

```bash
cat /etc/fstab
```

Finding:

```text
/dev/disk/by-uuid/caf05bcf-77f9-4d7b-b26a-dbe9ee1e5b65 / ext4 defaults 0 1
```

The UUID matched the value reported by `blkid`.

## Step 4: Verify the active root mount

Command:

```bash
findmnt /
```

Finding:

```text
TARGET  SOURCE           FSTYPE  OPTIONS
/       /dev/nvme0n1p2   ext4    rw,relatime
```

This confirmed that `/dev/nvme0n1p2` was the currently active root filesystem.

## Exercise conclusion

The three checks answered different questions:

1. `lsblk -f` identified the device and filesystem.
2. `blkid` provided the filesystem UUID.
3. `/etc/fstab` contained that UUID in the configured root entry.
4. `findmnt /` confirmed the active mount.

Together, these findings provided a consistent picture of the Ubuntu root filesystem.

---

# 8. Troubleshooting lessons

## 8.1 A disk is visible but has no listed partitions

If a disk appears in `lsblk` but has no child partition entries, it may have no partitions currently recognized by the system.

Useful read-only inspection commands include:

```bash
lsblk -f
sudo fdisk -l /dev/sdX
sudo sgdisk -p /dev/sdX
sudo blkid
```

Replace `/dev/sdX` with the correct device only after verifying its identity.

**Important:** Do not assume the disk is empty or safe to format. The absence of partitions in the current table does not establish that previous data is unrecoverable.

## 8.2 A filesystem is missing from `df -h`

`df -h` mainly reports mounted filesystems. An unmounted disk or partition may not appear there.

To investigate, compare:

```bash
lsblk -f
findmnt
```

The first identifies block devices and their filesystem information; the second lists active mounts.

## 8.3 The UUID in `/etc/fstab` does not match

A UUID mismatch can indicate that the configuration refers to an old filesystem, the wrong device, or a filesystem that has changed.

A careful investigation would compare:

```bash
sudo blkid
cat /etc/fstab
findmnt
```

Do not edit `/etc/fstab` blindly. An incorrect persistent mount configuration can cause boot delays or other boot-time problems.

## 8.4 The SMART report says `PASSED`, but errors appear in the logs

A `PASSED` assessment does not guarantee reliability.

Review the attributes, error logs, and self-test results together. Consider whether the errors are historical, recurring, or accompanied by other signs of trouble.

For my Seagate HDD, the historical uncorrectable errors were enough to make me avoid trusting it with important data.

---

# 9. Complete command cheat sheet

Use this section as a quick reference when returning to Linux storage work.

| Goal | Command | Explanation |
|---|---|---|
| List storage devices | `lsblk` | Shows disks, partitions, and device relationships |
| Show filesystems and UUIDs | `lsblk -f` | Adds filesystem type, UUID, and mount information |
| Check capacity and usage | `df -h` | Displays human-readable filesystem usage |
| Inspect filesystem identifiers | `sudo blkid` | Reports filesystem types, UUIDs, and partition identifiers |
| List disk partition information | `sudo fdisk -l /dev/sda` | Inspects the specified disk's partition table |
| Print GPT partition information | `sudo sgdisk -p /dev/sda` | Displays the GPT layout without requesting changes |
| Check package installation | `dpkg-query -W smartmontools` | Queries the installed package database |
| Check package availability | `apt-cache policy smartmontools` | Displays package version and repository information |
| Install SMART tools | `sudo apt install smartmontools` | Installs the diagnostic package |
| Inspect SMART information | `sudo smartctl -a /dev/sda` | Reads supported SMART data and logs |
| List current mounts | `findmnt` | Displays active filesystem mounts |
| Check the root mount | `findmnt /` | Shows the filesystem mounted at `/` |
| Read persistent mount configuration | `cat /etc/fstab` | Displays configured filesystem and swap entries |
| List fstab entries structurally | `findmnt --fstab` | Displays configured mount entries |

## Common flags and symbols

| Flag or symbol | Meaning |
|---|---|
| `sudo` | Run a command with elevated privileges |
| `-h` | Human-readable sizes for `df` |
| `-f` | Show filesystem details with `lsblk` |
| `-l` | List mode for `fdisk`; its meaning depends on the command |
| `-a` | Display all supported SMART information with `smartctl` |
| `-p` | Print GPT layout with `sgdisk` |
| `--fstab` | Read mount configuration from `/etc/fstab` with `findmnt` |
| `/` | Root directory and root filesystem mount point |
| `/dev/` | Directory containing device files |
| `/dev/sda` | Example disk device path |
| `/dev/sda1` | Example first partition on a disk |
| `/dev/disk/by-uuid/` | Stable device paths based on filesystem UUIDs |
| `#` | Starts a comment in `/etc/fstab` |
| `rw` | Read-write mount option |
| `relatime` | Performance-conscious access-time update policy |

A flag's meaning depends on the command it belongs to. Always consult the command's manual page if unsure.

For example:

```bash
man lsblk
man findmnt
man smartctl
man fstab
```

Press `q` to leave a manual page.

---

# 10. Storage safety checklist

Before working with an unfamiliar drive:

- [ ] Identify the device using `lsblk`.
- [ ] Confirm the model and capacity.
- [ ] Check partition and filesystem information.
- [ ] Determine whether the device is mounted.
- [ ] Check whether the disk may contain data that needs to be preserved.
- [ ] Inspect SMART information when supported.
- [ ] Verify the exact target device before any operation that changes it.
- [ ] Back up important data before making changes.
- [ ] Understand the consequences before formatting, repartitioning, or editing persistent mount configuration.

Commands that create filesystems, change partition tables, or write to disks should not be used casually. They can destroy data or make it harder to recover.

For this lab, I used inspection commands and did not format, repartition, or mount the 1 TB HDD.

---

# 11. Final findings and conclusions

## Ubuntu system storage

- Ubuntu is installed on the NVMe drive.
- The root filesystem is `/dev/nvme0n1p2`, formatted as `ext4`.
- The EFI partition is `/dev/nvme0n1p1`, formatted as `vfat`.
- The root filesystem UUID matched between `blkid` and `/etc/fstab`.
- `findmnt /` confirmed the active root mount.
- The root filesystem reported approximately 233G total capacity, 16G used, and 205G available in the observed `df -h` output.

## Windows-related storage

- `/dev/sdb` is an approximately 179 GB SATA SSD.
- It contains partitions reported as `vfat`, `ntfs`, and a Microsoft reserved partition.
- Those partitions were not mounted during the inspection.

## Additional HDD

- `/dev/sda` is a Seagate 1 TB HDD.
- GPT partition
