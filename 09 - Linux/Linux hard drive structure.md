Linux abstracts physical storage into a layered stack—from block devices and partition tables up to a single unified directory tree rooted at `/`.

## Storage Stack (Bottom to Top)

1. **Physical / Virtual Disks**: Exposed as block devices under `/dev` (`/dev/sd*`, `/dev/nvme*`, `/dev/vd*` — see [[Linux devices]]).
2. **Partition Table**: Divides a disk into partitions using **GPT** (modern standard paired with [[EFI|UEFI]], >2 TiB disks, 128+ partitions) or legacy **MBR** (2 TiB / 4 primary partition limit).
3. **Optional Volume / Encryption Layer**:
   - **LUKS / dm-crypt**: Full-disk encryption at rest (often [[AES256]]-XTS).
   - **LVM (Logical Volume Manager)**: Abstracts disks into pools via `/dev/mapper/*` so volumes can be resized or snapshotted online.
4. **Filesystem**: Formats a partition/volume (`ext4`, `XFS`, `Btrfs`, `ZFS`) with **inodes** (file metadata + block pointers) and **data blocks**.
5. **Mount Table (`/etc/fstab`)**: Attaches filesystems onto directories in the root tree.

In Linux LVM, the storage hierarchy from bottom to top is ==Physical Volume (PV) -> Volume Group (VG) -> Logical Volume (LV)==.

## Key Directories (Filesystem Hierarchy Standard)

- `/boot` & `/boot/efi`: Kernel (`vmlinuz`), `initramfs`, GRUB, and the FAT32 [[EFI]] System Partition (ESP).
- `/etc`: Host-specific system and service configuration files.
- `/var`: Variable runtime data—logs (`/var/log`), caches, queues, and [[Database]] files. Often placed on a separate partition so runaway logs cannot fill `/`.
- `/tmp`: Temporary files, commonly mounted on `tmpfs` (RAM-backed).
- `/dev`, `/proc`, `/sys`: Virtual kernel filesystems exposing devices, running processes, and kernel tunables (`sysctl`).
- `/usr`: Read-only user-space binaries, libraries, and shared data.
- `/home`: User directories.

> [!warning] Failure modes & trade-offs
> - **Inode exhaustion (`No space left on device` with free disk space)**: Creating millions of tiny files (e.g., cache/session files) can exhaust the filesystem's inode table (`df -i`) while block usage (`df -h`) still looks half-empty.
> - **Deleted-but-open files**: Running `rm` on a huge log file held open by a daemon removes only the directory entry; disk space is not reclaimed until the process closes the file descriptor. Find them with `lsof +L1`, or truncate in place (`: > app.log`).
> - **Single-partition `/` blast radius**: Putting `/var/log` and `/tmp` on `/` means a single noisy service or core dump can hit 100% disk and break SSH, [[Database]] writes, and [[Container]] runtimes.

Always mount disks in `/etc/fstab` by `UUID` or `PARTUUID` rather than `/dev/sdX` because kernel device enumeration order can change across reboots. #card-reminder

Why does `rm` on a large open log file not immediately free disk space?::The directory entry is removed, but the inode and data blocks remain allocated until every process holding an open file descriptor to it closes the file (inspect with `lsof +L1`).

## How can Linux report "No space left on device" when `df -h` shows plenty of free disk space? #card

Two common causes:
1. **Inode exhaustion**: The filesystem ran out of inodes due to millions of tiny files (check with `df -i`).
2. **Reserved root blocks**: `ext4` reserves ~5% of blocks for `root` by default, so non-root processes hit `ENOSPC` before `df -h` reaches 100%.

## Related

- [[Linux devices]]
- [[EFI]]
- [[SCSI]]
- [[Linux Terminal]]
- [[Container]]
- [[AES256]]
