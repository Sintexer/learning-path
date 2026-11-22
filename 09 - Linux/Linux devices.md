---
flashcards:
  q-aur4: { nid: 1795356716967, hash: 98ty9uge, sync: tfavkrsi }
  q-tdge: { nid: 1795356717001, hash: zjkhj39z, sync: f35jg35w }
  q-ma9b: { nid: 1795356717020, hash: tpdt8yez, sync: 2fbzciqm }
---

`lsscsi` (and `lsblk` / `blkid`) lists attached storage devices. Because the kernel assigns names like `/dev/sda` and `/dev/sdb` in discovery order, removing or adding a drive can shift device names across reboots. To avoid breakage, Linux identifies filesystems by immutable **UUIDs** (accessible under `/dev/disk/by-uuid/` and configured in `/etc/fstab` — see [[Linux hard drive structure]]).

Devices are exposed as special files under `/dev` in two main classes:
- **Block devices** (`b` in `ls -l /dev`): Random-access, kernel-buffered storage devices (e.g., `/dev/sda`, `/dev/nvme0n1`).
- **Character devices** (`c` in `ls -l /dev`): Unbuffered byte-stream devices (e.g., `/dev/tty`, `/dev/null`, `/dev/urandom`).

## Common Device Nodes under `/dev`

1. `/dev/sd*` — [[SCSI]] disks, also used for SATA and USB storage via the kernel SCSI subsystem. Three drives appear as `sda`, `sdb`, `sdc`; each partition is a numbered device (`sda1`, `sda2`).
2. `/dev/xvd*` and `/dev/vd*` — Paravirtualized disks exposed by [[Hypervisor|hypervisors]] (Xen and KVM/virtio) to [[Virtual machine|virtual machines]].
3. `/dev/nvme*` — NVMe SSDs over PCIe (e.g., `/dev/nvme0n1` for namespace 1, `/dev/nvme0n1p1` for partition 1).
4. `/dev/dm-*` and `/dev/mapper/*` — Device Mapper virtualization layer (used by LVM, LUKS disk encryption, and multipath); `/dev/mapper/*` provides human-readable symlinks to `/dev/dm-*`.
5. `/dev/sr*` — Optical drives (CD/DVD).
6. `/dev/hd*` — Legacy PATA (Parallel ATA / IDE) devices.
7. `/dev/tty*`, `/dev/pts/*` — Terminal devices: `/dev/tty` represents a process's controlling terminal, while `/dev/pts/*` are pseudoterminal (PTY) slaves used by terminal emulators and SSH sessions (see [[TTY]]).
8. `/dev/ttyS*`, `/dev/ttyUSB*`, `/dev/ttyACM*` — Serial ports (RS-232, USB-to-serial adapters, microcontrollers).
9. `/dev/lp0`, `/dev/lp1` — Parallel port devices (legacy printers).
10. `/dev/snd/*`, `/dev/dsp`, `/dev/audio` — Audio devices.
11. `/dev/sg*` — [[SCSI]] generic devices, allowing raw SCSI commands to be sent directly to a device when high-level block drivers do not suffice.

What is the difference between a block device and a character device in Linux `/dev`?::A block device (e.g., `/dev/sda`, `/dev/nvme0n1`) transfers fixed-size blocks with random access and kernel page-cache buffering, whereas a character device (e.g., `/dev/tty`, `/dev/null`, `/dev/urandom`) streams bytes sequentially without block buffering.
^q-aur4

In `/dev/nvme0n1p2`, `nvme0` is the ==controller==, `n1` is the ==namespace==, and `p2` is ==partition 2==.
^q-tdge

> [!tip] Production reality
> On cloud VMs (like AWS Nitro EC2 instances), EBS volumes attach as `/dev/nvme*` even if your Terraform config specifies `/dev/sdf`. Never hardcode `/dev/sd*` in automation—use `lsblk -f`, `blkid`, or `/dev/disk/by-id/` / `/dev/disk/by-uuid/`.

Why is referencing disks by `/dev/sdX` in `/etc/fstab` or scripts unsafe?::Because `/dev/sdX` names are assigned dynamically in kernel discovery order and can shift when disks are added, removed, or initialized in a different order across reboots—use `UUID` or `/dev/disk/by-id/` instead.
^q-ma9b

## Related

- [[Linux hard drive structure]]
- [[SCSI]]
- [[TTY]]
- [[EFI]]
- [[Hypervisor]]
- [[Virtual machine]]
