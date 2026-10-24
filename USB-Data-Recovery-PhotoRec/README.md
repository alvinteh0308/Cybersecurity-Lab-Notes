# USB Data Recovery Using PhotoRec — Digital Forensics Lab

## 1. Project Overview

This project demonstrates how to recover files from a formatted USB flash drive using Kali Linux, `dd`, and PhotoRec.

The objective is to understand disk imaging, file carving, filesystem metadata and data recovery while preserving the original USB data.

## 2. Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux (VMware) |
| USB Device | Silicon Power 8GB |
| Device Path | `/dev/sdb` |
| Partition | `/dev/sdb1` |
| File System | FAT32 |
| Disk Imaging | GNU dd |
| Recovery Tool | PhotoRec 7.2 |
| Supporting Tools | lsblk, script, file, sha256sum |

## 3. Methodology

### Step 1 — Start Terminal Session Logging

Before beginning the investigation, I used the `script` command to record my terminal session.

```bash
script -a session.log
```

**Command Explanation**

- `script`: Records terminal input and output into a log file.
- `-a`: Append mode. Adds new content to an existing log rather than overwriting it.
- `session.log`: The filename used to store the terminal recording.

This helps document the commands executed during the investigation for later review.

To stop recording:

```bash
exit
```

### Step 2 — Identify the USB Device

I connected the USB flash drive to my Kali Linux virtual machine and identified it using:

```bash
lsblk -o NAME,SIZE,MODEL,FSTYPE,MOUNTPOINTS
```

**Command Explanation**

- `lsblk`: Lists available block storage devices.
- `-o`: Specifies which columns to display.
- `NAME`: Device name.
- `SIZE`: Storage capacity.
- `MODEL`: Device model.
- `FSTYPE`: Filesystem type.
- `MOUNTPOINTS`: Current mount locations.

The USB was identified as `/dev/sdb`, containing a FAT32 partition at `/dev/sdb1`.

I confirmed that the USB partition was not mounted before imaging.

### Step 3 — Create a Forensic Disk Image

Instead of recovering files directly from the USB, I created a raw disk image using GNU `dd`.

```bash
sudo dd if=/dev/sdb of=usb.img bs=4M status=progress
```

**Command Explanation**

| Parameter | Meaning | Purpose |
|---|---|---|
| `sudo` | Superuser Do | Execute with administrator privileges |
| `dd` | Raw data copying utility | Copy disk data at the block level |
| `if=/dev/sdb` | Input File | Read data from the USB |
| `of=usb.img` | Output File | Save data to an image file |
| `bs=4M` | Block Size | Process 4 MiB per read/write block |
| `status=progress` | Progress display | Show bytes copied during imaging |

#### Why use `bs=4M`?

The `bs` parameter determines the block size used by `dd`.

In GNU `dd`, `4M` means 4 MiB, equivalent to 4,194,304 bytes.

For example, when copying an 8 GiB USB:

| Block Size | Approximate Number of Blocks |
|---|---:|
| 512 bytes | 16,777,216 |
| 1 MiB | 8,192 |
| 4 MiB | 2,048 |
| 16 MiB | 512 |

A larger block size generally reduces the number of read and write operations. However, a larger block size does not always mean faster performance.

I selected `bs=4M` as a practical balance between performance and buffer size.


### Step 4 — Recover Files Using PhotoRec

I launched PhotoRec using the disk image:

```bash
photorec usb.img
```

The following options were selected:

| Setting | Selection |
|---|---|
| Partition | FAT32 LBA |
| Filesystem Type | Other |
| Scan Mode | Whole |
| Destination | Recovered directory on Kali |

**Why select Whole instead of Free?**

- `Free` scans space marked as unallocated by the current filesystem.
- `Whole` scans the entire selected partition.

Since the USB had previously been formatted, I selected `Whole` to search for file signatures throughout the partition.


## 4. Technical Concepts Learned

### File System Metadata vs Actual Data

When a file is deleted or a USB is quick-formatted, filesystem metadata may be modified without immediately overwriting all original file content.

This means a file may no longer appear in Windows File Explorer even though some of its data remains on the storage device.

### File Carving

PhotoRec uses file carving to identify and reconstruct files from raw storage data.

For example, JPEG files commonly begin with the hexadecimal signature:

```text
FF D8 FF
```

PhotoRec uses known file signatures and format structures to attempt recovery without requiring the original filename or directory entry.

### Data Overwriting

Deleted data may become unrecoverable if new data overwrites the same storage locations. On USB flash storage, controller behaviour may also affect data persistence.

This is why preserving the original device and analysing a disk image is important.

## 5. Recovery Results

*To be completed with the actual PhotoRec recovery results.*

Document the number of files recovered, their formats, whether they opened successfully and any limitations observed.

## 6. Conclusion

This lab provided practical experience with Linux storage commands, raw disk imaging, terminal session logging and file recovery using PhotoRec.

It also demonstrated why deleted or formatted files may remain recoverable and why forensic imaging is useful for preserving source data during an investigation.

## 7. Ethical Considerations

All activities were conducted on an owned or authorised USB device for educational purposes. Raw disk images, recovered personal files and unredacted terminal logs are excluded from the public repository.