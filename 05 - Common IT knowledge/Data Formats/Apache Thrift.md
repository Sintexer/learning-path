Born in Facebook. Has 3 different binary encoding formats: BinaryProtocol, CompactProtocol, DenseProtocol (Only c++).


```thrift
struct Person {
	1: required string userName,
	2: optional i64 favoriteNumber,
	3: optional list<string> interests
}
```

Comes with code generation tool.