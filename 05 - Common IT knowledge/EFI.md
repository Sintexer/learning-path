---
flashcards:
  q-sy9p: { nid: 1795356724442, hash: aeiht4cv, sync: ua5phxzj }
  q-zpwi: { nid: 1795356724467, hash: 6mq4wa6b, sync: vtbf4y4q }
  q-mdrx: { nid: 1795356724492, hash: dfujc6at, sync: aifpjtxn }
---

**EFI (Extensible Firmware Interface)** and **UEFI (Unified Extensible Firmware Interface)** are software interfaces between an operating system and the firmware of a system. They are modern alternatives to the traditional **BIOS (Basic Input/Output System)** interface.

BIOS is a simple, low-level interface that dates back to the earliest IBM PC. When a computer with a BIOS starts up, the BIOS firmware initiates a Power-On Self Test (POST) and then locates the **Master Boot Record (MBR)**, where the bootloader resides. The BIOS then transfers control to the bootloader, which starts booting the operating system.

EFI/UEFI offers several advantages over BIOS:

1. **Support for GPT (GUID Partition Table)**: GPT partitions can exceed MBR's 2 TiB disk limit and 4-primary-partition limit (supporting 128+ partitions by default and storing a redundant backup partition table at the end of the disk — see [[Linux hard drive structure]]).
2. **Improved boot and load times**: UEFI initializes hardware in parallel and loads OS bootloaders directly from a dedicated FAT32 **EFI System Partition (ESP)** (typically mounted at `/boot/efi` in Linux).
3. **Compatibility with modern hardware and operating systems**: UEFI runs natively in 32-bit or 64-bit protected mode (vs BIOS's 16-bit real mode) and removes legacy addressing limits.
4. **Secure Boot**: UEFI supports **Secure Boot**, which cryptographically verifies that bootloaders and OS kernels carry trusted signatures before executing them, helping protect against boot-sector malware and rootkits.
5. **Better User Interface**: UEFI often provides a graphical setup interface with mouse/network support rather than legacy text-only BIOS menus.

In essence, EFI/UEFI is a comprehensive evolution of traditional BIOS, offering more features and broader compatibility with modern hardware and software.

What is the EFI System Partition (ESP)?::A dedicated FAT32 partition (commonly mounted at `/boot/efi` on Linux) where UEFI firmware reads OS bootloaders (e.g., GRUB, systemd-boot) and firmware updates.
^q-sy9p

Unlike BIOS/MBR (limited to ==2 TiB disks and 4 primary partitions==), UEFI/GPT supports ==64-bit LBA addressing (up to 9.4 ZB), 128+ partitions, and a backup partition table==.
^q-zpwi

> [!tip] Production reality
> On Linux workstations and servers, UEFI Secure Boot frequently blocks unsigned out-of-tree kernel modules (such as proprietary NVIDIA drivers or custom eBPF/dkms modules) unless you enroll a Machine Owner Key (MOK) or disable Secure Boot.

## How does UEFI Secure Boot protect the boot chain? #card

UEFI firmware stores trusted public keys in NVRAM (`PK`, `KEK`, `db`). At startup, it verifies the cryptographic signature of the bootloader (e.g., shim/GRUB), which in turn verifies the OS kernel signature before handing over execution—preventing unsigned bootkits or tampered kernels from running.
^q-mdrx

## Related

- [[Linux hard drive structure]]
- [[Linux devices]]
- [[SCSI]]
- [[Virtual machine]]
