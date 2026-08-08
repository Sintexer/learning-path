Consumer receives the data from the broker's topic (identified by name) using the *pull model*. Consumers automatically know which broker to read from. In case of broker failures, consumers know how to recover. 

Data is read from low to high offset **within each partition**.

## Consumer groups

All the consumers in an application read data as a *consumer group*. Each consumer within a group reads from exclusive partitions.