---
flashcards:
  q-s6v9: { nid: 1795356718091, hash: yvpyytxc, sync: g27imker }
  q-rb9y: { nid: 1795356718119, hash: ak3nzq2q, sync: spbkd2rg }
---

**SCSI** stands for **Small Computer System Interface**. It is a set of standards for physically connecting and transferring data between computers and peripheral devices.

SCSI is most commonly used for hard disk drives and tape drives, but it can connect a wide range of other devices, including scanners and optical drives. It has long been the standard method for connecting storage devices to servers and enterprise systems.

SCSI enables communication between the computer and peripheral devices by sending structured **SCSI commands** (CDBs — Command Descriptor Blocks). Each device class implements its own SCSI command set.

## Why Does Linux Use SCSI?

Linux, like other operating systems, needs a unified way to communicate with storage hardware. Even when Linux talks to non-SCSI physical buses (such as SATA drives, SAS drives, or USB flash drives), it exposes them through the kernel's **SCSI subsystem** (creating `/dev/sd*` and `/dev/sg*` nodes — see [[Linux devices]]):

- **High performance**: Designed for enterprise throughput and queued I/O.
- **Flexibility**: Handles a wide range of device types and supports multi-device topologies (including **iSCSI**, which encapsulates SCSI commands over [[TCP]]/IP for network-attached SAN storage).
- **Hot-pluggability**: Devices can be added or removed without shutting down the system.

Because the SCSI command set provides a rich, uniform abstraction layer, Linux uses **SCSI translation/emulation** (e.g., `libata` for SATA, `usb-storage` for USB) so upper storage layers do not need separate drivers for every drive interface.

Why do SATA and USB drives appear as `/dev/sd*` (SCSI disks) in Linux?::Linux routes SATA (`libata`) and USB mass storage through its kernel SCSI subsystem because SCSI provides a uniform, rich command abstraction for block storage devices.
^q-s6v9

## SCSI vs NVMe

While SCSI/SAS and SATA/AHCI were designed around spinning disks (shallow command queues), modern flash SSDs increasingly use **NVMe** directly over PCIe, offering ==up to 64K queues with 64K commands per queue== instead of a single narrow queue—dramatically lowering I/O latency and maximizing parallel IOPS.
^q-rb9y

> [!tip] Production reality
> On modern cloud hypervisors (e.g., AWS Nitro), attached block volumes (EBS) are exposed as NVMe devices (`/dev/nvme*`) rather than `/dev/sd*` or `/dev/xvd*`. Always mount filesystems by `UUID` or label in `/etc/fstab` rather than relying on device names ([[Linux hard drive structure]]).

## Related

- [[Linux devices]]
- [[Linux hard drive structure]]
- [[EFI]]
- [[TCP]]
- [[Virtual machine]]
