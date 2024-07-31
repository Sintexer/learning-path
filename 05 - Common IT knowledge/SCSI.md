SCSI stands for Small Computer System Interface. It's a set of standards for physically connecting and transferring data between computers and peripheral devices. 

SCSI is most commonly used for hard disk drives and tape drives, but it can connect a wide range of other devices, including scanners and CD drives. It has long been the standard method for connecting such devices to servers and other enterprise systems.

SCSI enables the communication between the computer and peripheral devices by sending SCSI commands. These devices are often disk drives, but can also be other types of devices, each with their own command sets.

Now coming to the question, why does Linux need SCSI?

Linux, like other operating systems, needs a way to communicate with hardware, and that's where SCSI comes in. While Linux can use other interfaces for connecting to devices (such as SATA for hard drives or USB for external devices), SCSI has certain advantages and features that make it useful in a variety of settings:

- High performance: SCSI is designed for speed and can handle high data transfer rates, which is crucial for applications like high-speed storage or real-time data processing.

- Flexibility: SCSI can handle a wide range of device types and supports daisy-chaining, i.e., having multiple devices connected in a series.

- Pluggability: Devices can be added or removed without shutting down the system, which is known as hot-plugging.

It's also worth noting that, in Linux, certain types of devices (such as SATA drives, SAS drives, or even USB flash drives) are managed as if they were SCSI devices, due to the 'SCSI emulation'. This is largely because the SCSI command set provides a rich, high-level interface for interacting with drives, compared to older interfaces like IDE.

In conclusion, SCSI is an important part of how Linux interacts with hardware, facilitating communication with a wide range of devices, particularly storage devices.