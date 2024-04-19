# JVM args

## Memory and heap related

- `-Xss` - stack size: 140M|1g...
- `-Xms` - initial heap size
- `-Xmx` - maximum possible heap size
- `-XX:MinHeapFreeRatio` - Min heap space percentage for every heap zone. If free space in a zone is less then this percentage, that additional space will be attached to this zone.
- `-XX:MaxHeapFreeRatio` - Max busy heap space percentage for every zone. If exceeded - additional space will be allocated for a zone.
- `-XX:NewRatio` - 1 new to x old space ratio. E.g. 3 means new space is 1/3 to the old gen space
- `-XX:NewSize` - New gen size.
- `-XX:MaxNewSize` - Max new gen size
- `-XX:Xmn` - Equivalent to `-XX:NewSize=x`, `-XX:ManNewSize=x`
- `-XX:SurvivorRatio` - Ratio of eden space to survivor. E.g. 6 means eden is 6 times larger than 1 survivor and eden is 1/8 of new gen.
