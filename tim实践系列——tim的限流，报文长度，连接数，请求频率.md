> 前言：[tim是去中心化分布式即时通讯引擎](https://tlnet.top/tim)。不依赖于任何中心服务器，采用去中心化分布式架构，解决传统中心化通讯方式的问题，去中心化分布式架构的通讯引擎的各个节点之间相互连接，形成一个庞大的分布式网络。可以轻松地扩展服务规模，支持更多的用户和业务需求，提供更加安全、可靠、高效的通讯服务。
> Github系列开源文章《[tim实践系列文章](https://github.com/donnie4w/Tim-Practical-Article)》

#### tim服务器限流，限制消息体大小，连接数，发信频率主要作用
1. 资源过载：当系统接收到超出其处理能力的大量请求时，如果不加以限制，可能会导致服务器内存溢出、CPU使用率过高等问题，进而引发服务崩溃或响应速度骤降。
2. 服务质量下降：持续的高流量可能使得正常用户的请求得不到及时响应，影响用户体验，如API接口响应超时等。
3. 带宽耗尽：对于网络传输而言，大量的数据传输可能导致带宽瞬间被消耗殆尽，从而影响整个网络的性能。
4. 数据库压力过大：针对数据库访问，限流可以有效避免大量并发读写操作对数据库造成冲击题。
5. 防止恶意请求流量，防止恶意攻击：限流可以限制恶意请求的流量，防止恶意攻击对系统造成损害。

#### 基于im本身的稳定性，安全性，可用性等考虑，tim提供了必要的安全支持
1. tim节点最大连接数
2. 连接请求频率限制
3. 请求报文最大长度限制

说明：

* 每个tim节点都可以配置节点的最大连接数。默认值为100万。超过连接限制的连接会直接被断开。
* 连接请求频率限制，指的是同一个用户连接每秒中请求数的最大值。默认没有限制。一般用户正常操作时，每秒钟的请求数应该在10以下，即使快速点击请求，也很难操作这个数。 在被服务器受到攻击的情况下，可能瞬间请求会飙升到几百上千，对节点带宽，CPU，后端数据库都会造成巨大压力，可能直接导致服务器崩溃。所以线上部署时，可以通过配置限制用户的请求频率，超过频率时，连接会被断开。
* 请求报文最大长度限制在正式环境中也是有必要的。正常应用会在用户输入或提交内容时，判断内容长度是否过长。但是在恶意攻击的情况下，用户可能通过其他手段绕过应用的判断，可以直接把数据提交到tim服务器，过长的报文可能导致带宽，数据库等服务压力过大。

以下是tim 限制参数的配置

    {
    "connectLimit":1000000,   
    "security":{
        "maxdata":8388608,     :
        "reqhzsecond":10
       }
    }

说明：

* "connectLimit":1000000  表示连接上限100万
* "maxdata":8388608  表示数据包最大长度：8M
* "reqhzsecond":10 表示同一个用户请求频率上限 10次/秒
