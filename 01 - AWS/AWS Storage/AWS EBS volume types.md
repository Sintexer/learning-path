Generally, [[AWS EBS|EBS]] could be **SSD** or **HDD** backed.:

- SSD: *gp3, gp2, io1, io2.* 
- HDD: *st1, sc1.*

SSDs are used for transactional workloads with frequent read/write operations with small I/O size. HDDs are used for large streaming workloads that need high throughput performance.  AWS offers two types of each.

**Use cases and explanation:**

- `gp2` and `gp3` (General Purpose `SSD`) balance price and performance. Are ideal for use cases such as boot volumes, medium-size single instance databases, and development and test environments.
- `io1` and `io2` (Provisioned IOPS `SSD`) support up to 64,000 IOPS and 1,000 MiB/s of throughput. This enables you to predictably scale to tens of thousands of IOPS per EC2 instance.
- `st1` (Throughput Optimized `HDD`) low-cost storage that defines performance in terms of throughput rather than IOPS. These volumes are ideal for large, sequential workloads such as Amazon EMR, ETL, data warehouses, and log processing.
- `sc1` (Cold HDD) provide low-cost magnetic storage that defines performance in terms of throughput rather than IOPS. These volumes are ideal for large, sequential, cold-data workloads. If you require **infrequent access** to your data and are looking to save costs, these volumes provide inexpensive block storage.

**SSD Volumes:**

|                                   | **General Purpose**   <br>**SSD volumes**     |                                               | **Provisioned IOPS**   <br>**SSD volumes**             |                                                        |                                                        |
| --------------------------------- | --------------------------------------------- | --------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| **Volume type**                   | **gp3**                                       | **gp2**                                       | **io2 Block Express**                                  | **io2**                                                | **io1**                                                |
| **Description**                   | for a wide variety of transactional workloads | for a wide variety of transactional workloads | designed for latency-sensitive transactional workloads | designed for latency-sensitive transactional workloads | designed for latency-sensitive transactional workloads |
| **Volume size**                   | 1 GiB–16 TiB                                  |                                               | 4 GiB–64 TiB                                           | 4GiB–16 TiB                                            | 4GiB–16 TiB                                            |
| **Max IOPS**   <br>**per volume** | 16,000                                        | 16,000                                        | 256,000                                                | 64,000                                                 | 64,000                                                 |
| **Max throughput per volume**     | 1,000 MiB/s                                   | 250 MiB/s                                     | 4,000 MiB/s                                            | 1,000 MiB/s                                            | 1,000 MiB/s                                            |
| **Amazon EBS Multi-attach**       | Not supported                                 | Not supported                                 | Supported                                              | Supported                                              | Supported                                              |

**HDD Volumes:**

|                               | **Throughput Optimized HDD volumes**                                            | **Cold HDD volumes**                                                |
| ----------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Volume type**               | **st1**                                                                         | **sc1**                                                             |
| **Description**               | A low-cost HDD designed for frequently accessed, throughput-intensive workloads | The lowest cost HDD designed for less frequently accessed workloads |
| **Volume size**               | 125 GiB–16 TiB                                                                  | 125 GiB–16 TiB                                                      |
| **Max IOPS per volume**       | 500                                                                             | 250                                                                 |
| **Max throughput per volume** | 500 MiB/s                                                                       | 250 MiB/s                                                           |
| **Amazon EBS Multi-attach**   | Not supported                                                                   | Not supported                                                       |