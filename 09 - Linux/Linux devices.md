`lssci` is the tool to list all devices. Sometimes it is inconvinient that devices are identified in the order kernel has recognized them. It brings some pain when one device is removed, and other devices might change their name. So Linux now uses UUID for devices, and stores UUID mapping in a special table.

Devices are visible as folder under /dev.

1. `dev/sd*` - these are SCSI disks. Used for SATA and USB devices. If you have three devices, they will be named sda, sdb, sdc. If one device has partiotions, each partition is a separate device (`sda1, sda2`)
2. `dev/xvd*` and `dev/vd*` - these are virtual disks that are used by hypervisors.
3. `dev/nvme` - for long-term storage devices like NVME SSDs.
4. `dev/dm-*` and `dev/mapper` - some systems use a special virtualization layer, so the programs use some device mapper to access devices. Usually `dev/dm` devices are symbolic links to `dev/mapper`.
5. `dev/sr*` - for CD and DVD
6. `dev/hd*` - PATA (parallel ATA) devices, sometimes SATA devices might appear here if they are running in PATA compatibility mode.
7. `dev/tty*`, `dev/pts/*` - are both representations of terminal devices, with `/dev/tty` representing an abstract controlling terminal for a process, and `/dev/pts` being used for each pseudoterminal, emulating physical terminals for things like terminal emulator windows and remote logins (see [[TTY]])
8. `dev/ttyS*`, `dev/ttyUSB*`, `dev/ttyASM*` - for outdated serial ports like RS-232.
9. `dev/lp0`, `dev/lp1` - one way parallel port devices, like used for printing pages
10. `dev/snd/*`, `dev/dsp`, `dev/audio`, etc. - audio devices
11. `dev/sg*` - is for [[SCSI]] generic devices. These are abstract representations of other devices, but with a SCSI plugged-in. Could be used to send SCI commands to a device if high-level set of command doesn't suit you.