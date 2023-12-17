> 前言:tim是去中心化的分布式im引擎。 传输协议是tim能架设支持百亿级别在线用户量的重要因素，tim的协议在协议包体积与协议序列化反序列化方面，都有非常优秀的表现。
> 来源[《tim实践系列》](https://github.com/donnie4w/Tim-Practical-Article)

tim协议支持两种格式，thrift 压缩格式与json格式；主要支持的协议使用gothrift进行序列化与反序列化.

** 测试数据与长度 **

| 字段名 | 数据 |长度(字节) |
| ----- | ----- |----- |
| MsType | 1 |1 |
| OdType| 1 |1 |
| Id | 1<<60 |8 |
| Mid | 1<<61 |8 |
| BnType | 1<<30 | 4 |
| FromTid | TidBean{Node: "tom"}|3|
| ToTid|TidBean{Node: "jerry"}|5 |
| Body | []byte("A good programmer")|17|
| IsOffline | true |1|
| Timestamp | 1<<60|8|
| Extend | map[string]string{"ex1": "A good programmer", "ex2": "A good programmer"}|40 |

**原始输入数据的总长度为96字节，各个框架序列化后数据长度：**

|  | 	序列化后数据长度|体积 |
| ----- | ----- |----- |
| Msgpack| 254 字节| 较大|
| Json    |	366 字节 |大 |
| Thrift  |	124 字节 |	小 |
|ProtoBuf |	125字节 | 	小|


----------
### 序列化反序列化 压测结果


#### 1.串行压测结果

| 序列化框架 | 运行数 | 效率(纳秒/次)	 | 分配内存 | 内存分配次数 |
| ----- | ----- |----- |----- |----- |
| MsgpackEncode-4 | 883100 |1411 ns/op | 616 B/op|8 allocs/op |
| MsgpackEncode-8 | 888198 |1400 ns/op | 616 B/op|8 allocs/op |
|  jsonEncode-4| 741568 |1622 ns/op |712 B/op |9 allocs/op |
|jsonEncode-8  |  703032|1694 ns/op |712 B/op |9 allocs/op |
| ThriftEncode-4 | 980503 |	1167 ns/op |456 B/op |7 allocs/op |
| ThriftEncode-8 | 1028618 |1186 ns/op |456 B/op |7 allocs/op |
| ProtoBufEncode-4 | 997920 | 	1234 ns/op|256 B/op |9 allocs/op |
|ProtoBufEncode-8  | 1000000 | 	1234 ns/op|256 B/op |9 allocs/op |

| 反序列化框架 | 运行数 | 效率(纳秒/次)	 | 分配内存 | 内存分配次数 |
| ----- | ----- |----- |----- |----- |
|MsgpackDecode-4 |526042 |2206 ns/op |432 B/op |28 allocs/op |
|MsgpackDecode-8 |515901 |2192 ns/op |432 B/op |28 allocs/op |
|jsonDecode-4 |220232 |	5560ns/op |536 B/op |22 allocs/op |
|jsonDecode-8 |215497 |5564ns/op |536 B/op |22 allocs/op |
|ThriftDecode-4 |879506 |1449 ns/op |912 B/op |22 allocs/op |
|ThriftDecode-8 |773559 |1548 ns/op |912 B/op |22 allocs/op |
|ProtoBufDecode-4 |696997 |1669 ns/op |928 B/op |29 allocs/op |
|ProtoBufDecode-8 |696936 |1737 ns/op |928 B/op |29 allocs/op |

### 压测结果比较：
| 序列化框架 |序列化 | 效率| 反序列化 | 效率|
| ----- | ----- |----- |----- |----- |
|Msgpack |1400 ns/op  |快 |2206 ns/op |快 |
|Json |1622 ns/op  |快 |5560 ns/op |	慢 |
|Thrift |1167 ns/op |很快 |1449 ns/op |很快 |
|ProtoBuf |1234 ns/op |很快 |	1669 ns/op |很快 |

----------
### 2.并行压测结果
| 序列化框架 | 运行数 | 效率(纳秒/次)	 | 分配内存 | 内存分配次数 |
| ----- | ----- |----- |----- |----- |
|MsgpackEncode-4  |2258724  |492.1 ns/op | 616 B/op|8 allocs/op |
|MsgpackEncode-8  |2138186  |557.9 ns/op |616 B/op |8 allocs/op |
|jsonEncode-4  |2028855  |582.4 ns/op |712 B/op |9 allocs/op |
|jsonEncode-8  |1777850  |686.1 ns/op |712 B/op |9 allocs/op |
|ThriftEncode-4  |2622094  |456.3 ns/op|456 B/op | 7 allocs/op|
|ThriftEncode-8  |2328318  |518.0 ns/op |456 B/op |7 allocs/op |
|ProtoBufEncode-4  |1745168  |624.9 ns/op |256 B/op |9 allocs/op |
|ProtoBufEncode-8  |1854494  |652.8 ns/op |256 B/op |9 allocs/op |


| 反序列化框架 | 运行数 | 效率(纳秒/次)	 | 分配内存 | 内存分配次数 |
| ----- | ----- |----- |----- |----- |
|MsgpackDecode-4  |1230190  |944.8 ns/op |1015 B/op |34 allocs/op |
|MsgpackDecode-8  |1000000  |1159ns/op |1015 B/op |34 allocs/op |
|jsonDecode-4  |578404 |2079  ns/op |1096  B/op |27 allocs/op |
|jsonDecode-8  |595042  |2381 ns/op |1096  B/op |27 allocs/op |
|ThriftDecode-4  |1905132  |756.4  ns/op |912  B/op |22 allocs/op |
|ThriftDecode-8  |1000000  |1103  ns/op |912  B/op |22 allocs/op |
|ProtoBufDecode-4  |1287536 |906.7  ns/op|928  B/op|29 allocs/op |
|ProtoBufDecode-8  |1000000 |1042 ns/op |928  B/op |29 allocs/op |

### 压测结果比较：
|  | 	序列化 | 效率	 | 反序列化| 效率 |
| ----- | ----- |----- |----- |----- |
| Msgpack | 492.1 ns/op  |很快 |944.8 ns/op |快 |
| Json | 582.4 ns/op   |快 |2079 ns/op| 慢|
| Thrift | 456.3 ns/op |很快 |756.4 ns/op |很快|
| ProtoBuf | 624.9 ns/op |快 |906.7 ns/op | 快|


----------
### **结论**：
tim的通讯协议主要数据类型是：数字，字节数组，字典，字符串

thrift与protobuf的数据压缩后体积相差无几，如果去掉字典类型，protobuf<thrift ,也是相差几个字节，它们都使用zagzig实现整数变长编码，protobuf的数字类型序列化效率更高。

由协议序列化的测试结果可以看出，在可能的条件下，应避免使用json协议，json协议的数据包体积与序列化效果都比较差。

说明：这个测试只是特定数据类型下的测试结果。并非说thrift的序列化性能高于protobuf。比如序列化数字类型，protobuf的序列化效率要高于thrift。这是针对tim所需的数据类型。

tim的序列化反序列化使用gothrift，具体可以参考文章 《gothrift 一 go版thrift性能优化项目》

比较有代表性的框架的序列化效果与性能在测试数据中已经详细说明，tim的序列化性能会在gothrift项目中继续进行优化改进。

tim支持在thrift与json之间的协议相互通信与转换，所以，基本所有图灵完备的编程语言都可以实现tim的客户端并接入tim服务进行通信。





[测试程序源码及测试结果](https://github.com/donnie4w/test/tree/main/timtest)

----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！