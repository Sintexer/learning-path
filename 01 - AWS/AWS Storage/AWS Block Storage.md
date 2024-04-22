Service examples: [[AWS EBS]], [[AWS EC2 Instance store]]

[[AWS File Storage|File storage]] treats files as a singular unit, but **block storage** splits files into fixed-size chunks of data called **blocks** that have their *own addresses*. Each block is an individual piece of data storage. Because each block is addressable, **blocks can be retrieved efficiently**. Think of block storage as a more direct route to access the data.  
  
When data is requested, the addresses are used by the storage system to organize the blocks in the correct order to form a complete file to present back to the requestor. Besides the address, no additional metadata is associated with each block.

If you want to change one character in a file, you just change the block, or the piece of the file, that contains the character. This ease of access is why block storage solutions are fast and use less bandwidth.

Because block storage is optimized for low-latency operations, it is a preferred storage choice for high-performance enterprise workloads and transactional, mission-critical, and I/O-intensive applications.

**Use cases:**

- Transactional workloads
- Containers
- Virtual machines
