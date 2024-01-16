> 前言：[tim是去中心化分布式即时通讯引擎](https://tlnet.top/tim)。不依赖于任何中心服务器，采用去中心化分布式架构，解决传统中心化通讯方式的问题，去中心化分布式架构的通讯引擎的各个节点之间相互连接，形成一个庞大的分布式网络。可以轻松地扩展服务规模，支持更多的用户和业务需求，提供更加安全、可靠、高效的通讯服务。
> Github系列开源文章《[tim实践系列文章](https://github.com/donnie4w/Tim-Practical-Article)》

tim支持使用各种关系型数据库，如Mysql，PostgreSQL ，SQL Server，Oracle，Oceanbase；tim默认数据库为TLDB；

#### tim支持除了tldb之外的数据库是非常有必要的

1. Mysql，Oracle 等系列的数据库都是互联网最常用的数据库，具备成熟的数据解决方案，是商用系统比较好的选择。
2. tim作为即时通讯引擎，必须能脱离自建的用户系统，并支持外部用户系统，提供更加无缝、高效且安全的通信服务。

#### tim支持外部用户系统的作用主要有：

1. 统一身份认证与权限管理：许多企业或组织已经有成熟的用户管理系统，包括用户账号、密码、角色权限等
2. 数据一致性与完整性：用户信息、联系人关系、组织架构等数据通常已经在企业的主数据库中维护。IM支持外部用户系统，可以避免信息孤岛和数据冗余。
3. 降低运维成本：维护一套独立的用户体系会增加系统的复杂性和运维成本

使用除了默认数据库TLDB之外的其他数据库，标识着使用外部的用户系统，因此，用户注册，用户资料维护，用户关系的维护，群组的建立与维护都在业务系统中自行完成，tim只专注于通讯部分的功能。

tim通过配置文件连接数据库，以mysql为例，加入配置文件为tim.json

    "sql.property": {
        "tim.mysql.connection": "root:123@tcp(127.0.0.1:3306)/timdb",
        "tim.sql.login": "SELECT uid FROM timuser WHERE username=? AND pwd=?",
        "tim.sql.token": "SELECT uid FROM timuser WHERE username=? AND pwd=?",
        "tim.sql.message.save": "insert into timmessage(rid,stanza,createtime) values(?,?,NOW())",
        "tim.sql.message.get": "SELECT id,stanza FROM timmessage WHERE rid=? AND id<? LIMIT 0,?",
        "tim.sql.message.get.byid": "SELECT stanza FROM timmessage WHERE id=?",
        "tim.sql.message.del.byid": "DELETE FROM timmessage WHERE id=?",
        "tim.sql.offlinemsg.save": "INSERT INTO timoffline (uuid,uniqueid,stanza,mid)VALUE(?,?,?,?)",
        "tim.sql.offlinemsg.save.mid": "INSERT INTO timoffline (uuid,uniqueid,mid)VALUE(?,?,?)",
        "tim.sql.offlinemsg.save.nomid": "INSERT INTO timoffline (uuid,uniqueid,stanza)VALUE(?,?,?)",
        "tim.sql.offlinemsg.get": "SELECT id,stanza FROM  timoffline WHERE uuid=? LIMIT 0,?",
        "tim.sql.offlinemsg.del.in": "DELETE FROM timoffline WHERE id IN (?)",
        "tim.sql.user.auth": "SELECT 1 FROM timroster WHERE fromuid=? AND touid=? and status=1",
        "tim.sql.user.exist": "SELECT 1 FROM timuser WHERE uid=?",
        "tim.sql.room.auth": "SELECT 1 FROM timroommember WHERE gid=? AND uid=? and status=1",
        "tim.sql.room.exist": "SELECT 1 FROM timgroup WHERE gid=?",
      }

说明：sql.property为关系型数据库配置的字段名称

sql.property中各属性字段的作用：

|属性字段 | 说明 |
| ----- | ----- |
| tim.mysql.connection | mysql数据源连接 格式：username:password@tcp(ip:port)/database |
| tim.sql.login | 有两个参数,第一个参数为用户名，第二个参数为密码，sql返回一个字段，用户标识，可以是id，或其他标识用户的字段   |
| tim.sql.token | 有两个参数,第一个参数为用户名，第二个参数为密码，sql返回一个字段，用户标识，可以是id，或其他标识用户的字段   |
| tim.sql.message.save | 保存聊天信息，有两个参数，第一个参数为64位整型类型，第二个参数为字节数组类型 |
| tim.sql.message.get | 获取聊天信息，3个参数，第一个参数对应tim.sql.message.save中第一个参数，第二个参数为信息id上限，第三个参数为查询结果条数，结果返回信息id与tim.sql.message.save第二个字段的字节数组 |
| tim.sql.message.get.byid | 根据id查询聊天信息，参数为信息id |
|tim.sql.message.del.byid  |根据id删除聊天信息，参数为信息id  |
| tim.sql.offlinemsg.save |保存离线信息，4个参数，第一个参数为64位整型，第二个参数为64位整型，第三个参数为字节数组，第四个参数为64位整型。配置该属性，将保存消息数据  |
|tim.sql.offlinemsg.save.mid  | 是保存离线信息的另一种选择，这是mid>0的情况，第一个参数为64位整型，第二个参数为64位整型，第三个参数为64位整型。即不保存消息体数据。 |
| tim.sql.offlinemsg.save.nomid | 该属性与tim.sql.offlinemsg.save.mid同时配置，第一个参数为64位整型，第二个参数为64位整型，第三个参数为字节数组。 |
|tim.sql.offlinemsg.get|获取离线信息，有两个参数，第一个参数对应tim.sql.offlinemsg.save第一个参数，第二个参数为返回条数，结果返回两个字段，第一个是离线信息表的信息id，第二个位消息体  |
| tim.sql.offlinemsg.del.in | 批量删除离线信息,该配置服务器做了。根据id删除离线信息表的数据，编写sal时，条件语句只需要写  in(?) 即可，无需写具体的参数个数，服务器根据删除的id数，会自动重新构建条件语句，如 in(?,?,?,?) |
| tim.sql.user.auth | 该配置为用户到用户的发信权限查询。有两个参数，第一个参数为消息发送者标识，第二个参数为消息接收者标识，返回1时，表示有权限发送，返回空行时或非1值时，表示无权限发送 |
|tim.sql.user.exist  | 该配置为查询账号表示是否存在，参数为登录或注册时sql返回的用户标识值。查询返回1时，表示用户存在，返回空行时或非1值时，表示无权限发送 |
| tim.sql.room.auth | 该配置为用户到群的发信权限查询。有两个参数，第一个参数为群标识，第二个参数为消息发送者标识，返回1时，表示有权限发送，返回空行时或非1值时，表示无权限发送 |
| tim.sql.room.exist | 该配置为查询群账号表示是否存在。查询返回1时，表示群存在，返回空行时或非1值时，表示不存在|

#### 不同数据库连接数据源

| 连接数据源 | 配置项 |示例 | 
| ----- | ----- | ----- |
|Mysql  |tim.mysql.connection  |root:123456@tcp(127.0.0.1:3306)/tim |
|postgreSQL  |tim.postgreSQL.connection  |host=127.0.0.1 port=5432 user=root password=123456 dbname=tim sslmode=disable |
|oracle  | tim.oracle.connection |	user="root" password="123456" connectString="127.0.0.1:1521/tim" |
|Sqlserver  | tim.sqlserver.connection | server=127.0.0.1;port=1433;userid=root;password=123456;database=tim|
|OceanBase | 	tim.mysql.connection | root:123456@tcp(127.0.0.1:3306)/tim|

说明：不同的数据库对应不同的配置项，OceanBase是采用连接mysql的方式。
由于不同驱动的实现，连接数据源的格式不同，开发者可以参考go连接数据库的格式。

tim提供了一种统一的模式连接数据库

#### 不同数据库连接数据源

|连接数据源 | 配置项 |
| ----- | ----- |
| Mysql | tim.mysql.connection.mod |
| postgreSQL | tim.postgreSQL.connection.mod |
|oracle |tim.oracle.connection.mod |
|Sqlserver|tim.sqlserver.connection.mod |
|OceanBase |tim.mysql.connection.mod |

tim.数据库.connection.mod  为tim提供统一格式的属性名形式，格式如下：
**host=IP地址   port=端口   user=用户名  password=密码  dbname=数据库名**

    如：host=127.0.0.1 port=5432 user=root password=123456 dbname=tim

#### 配置文件示例：
    "sql.property": {
        "tim.mysql.connection.mod": "host=127.0.0.1 port=3306 user=root password=123456 dbname=tim",
        ....
     }

开发者可以采用mod的统一模式，设置数据库连接源，如果连接源需要设置其他参数，则需使用原生连接 "tim.数据库.connection"