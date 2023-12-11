> tim 定义了TimMessage，TimPrecence，TimStream三种数据对象传递数据。其中支持持久化的对象为TimMessage，所以，一般用于用户之间的通信信息，系统信息，此外，TimMessage还支持部分tim的业务信息，如，加好友，入群的信息通知，与部分流数据传递。

#### TimMessage的字段：

| 表列 A | 数据类型 |是否必填 |作用 |值说明 |
| ----- | ----- | ----- | ----- | ----- |
| msType |	8位整型  |是  |消息类型  |1.系统至用户  2.用户至用户  3.用户至群  |  
| odType | 8位整型 | 是 | 	指令类型 |1.常规发信 .2,撤回信息,  3焚烧信息  .4业务处理   5流数据，其他>30 开发者自定义，小于30为 tim 系统 保留值。 | 
|id |64位整型 |否 |timMessage包id |	id值标识每一个timMessage包，避免终端重复接收协议包 |
|mid |64位整型 |否 | 保存到数据库的消息id| 常规发信持久化后的消息id|
|bnType   |32位整型 |否 |业务类型 |这是tim内置用户系统的业务类型，对应odType=4的情况下，tim的非核心接口类型 BUSINESS_ROSTER  = 1拉取花名册BUSINESS_USERROOM = 2拉取群账号  BUSINESS_ROOMUSERS= 3拉取群成员账号 BUSINESS_ADDROSTER = 4加好友  BUSINESS_FRIEND  = 5成为好友BUSINESS_REMOVEROSTER = 6删除好友  BUSINESS_BLOCKROSTER = 7拉黑账号  BUSINESS_NEWROOM  = 8建群  BUSINESS_ADDROOM = 9加入群  BUSINESS_PASSROOM  = 10通过加群申请  BUSINESS_NOPASSROOM = 11不通过加群申请  BUSINESS_PULLROOM  = 12拉人入群  BUSINESS_KICKROOM = 13踢人出群   BUSINESS_BLOCKROOM = 14拉黑群  BUSINESS_BLOCKROOMMEMBER = 15拉黑群成员   BUSINESS_LEAVEROOM  = 16退群  BUSINESS_CANCELROOM  = 17注销群  BUSINESS_BLOCKROSTERLIST = 18拉取账号黑名单   BUSINESS_BLOCKROOMLIST  = 19拉取账号拉黑群名单   BUSINESS_BLOCKROOMMEMBERLIST= 20管理员拉取群黑名单 |
| fromTid|Tid对象	 |否 |发送者 |node用户账号domain域名resource资源termtyp类型 extend扩展 |
|toTid |Tid对象 |否 |发送目标用户对象 |node用户账号domain域名resource资源termtyp类型 extend扩展 |
|roomTid |Tid对象 |否 |发送群对象 |node群账号domain域名resource资源termtyp类型 extend扩展 |
|dataBinary |字节数组 |否 |发送流数据消息体 |主要消息体 |
|dataString |字符串	 |否 |发送字符串消息体 | 主要消息体|
|isOffline |布尔值	 |否 |是否离线	 | |
|timestamp |64位整型	 |否 |消息时间	 |时间戳精确到纳秒 |
|udtype |16位整型	 | 否| 扩展字段	|开发者自定义：消息类型：图文，视频，音频等|
|udshow |16位整型	 |否 |扩展字段	 |开发者自定义：显示，如：显示类型：普通信息，系统信息，提示系统等 |
|extend |字典 |否	 |扩展字段 |字典类型，键值对，键字符串  值字符串 |
| extra|字典 |否	 |扩展字段 |字典类型，键值对，键字符串  值字节数组|

**开发者使用TimMessage时，需要对timMessage进行赋值，一些字段不需要客户端赋值的，包括id，fromTid，isOffline，timestamp  这些字段是由服务器赋值发送到客户端，所以客户端对这些字段赋值是无效的。**
## 以下是赋值字段的说明：
**msType** 是TimMessage必须确定值的字段

msType类型只有3个值，客户端只能使用msType=2，msType=3 两个值，msType=1表示系统信息，客户端如果设置msType=1，服务会返回参数错误。

msType=2表示 信息是发送到用户，msType=2表示 信息是发送到群。

msType=2时，如果同时设置ToTid与RoomTid，即该信息是一条群私信，表示信息发送到同一个群中的群成员。

**odType** 是TimMessage必须确定值的字段

odType 是8位整型，可以设置256个值，其中负数及小于等于30正数是tim保留值，开发者可以自定义>30的其他整型

**odType** 表示消息指令，timMessage传递tim的通信消息及指令

1.常规发信 .2,撤回信息,  3焚烧信息  .4业务处理   5流数据
odType=5时，timMessage为流数据，流数据只能发送到在线用户，如果用户离线，消息会被抛弃。流数据也不会持久化。流数据适合用于音视频等实时数据。

odType=2时，表示将撤回指定的Mid消息，客户端收到该数据的TimMessage时应该相应删除客户端中存储的消息。

odType=3时，由消息接收者发起的timMessage消息，为消息阅后即焚。

odType=4时，为tim内置的用户系统接口，如果不使用tim内置用户系统接口，该值无用。如果调用了tim内置用户系统接口，tim会在TimMessage中，会指定odType=4，并指定bnType  值告知客户端消息类型，如：加好友，成为好友，申请入群等



**bnType** 表示tim内置的用户系统业务接口。

这是tim内置用户系统的业务类型，对应odType=4的情况下，tim的非核心接口类型
1. BUSINESS_ROSTER  = 1  拉取花名册
2. BUSINESS_USERROOM = 2  拉取群账号
3. BUSINESS_ROOMUSERS= 3  拉取群成员账号
4. BUSINESS_ADDROSTER = 4  加好友
5. BUSINESS_FRIEND  = 5  成为好友
6. BUSINESS_REMOVEROSTER = 6  删除好友
7. BUSINESS_BLOCKROSTER = 7  拉黑账号
8. BUSINESS_NEWROOM  = 8  建群
9. BUSINESS_ADDROOM = 9  加入群
10. BUSINESS_PASSROOM  = 10 通过加群申请
11. BUSINESS_NOPASSROOM = 11 不通过加群申请
12. BUSINESS_PULLROOM   = 12 拉人入群
13. BUSINESS_KICKROOM = 13 踢人出群
14. BUSINESS_BLOCKROOM = 14 拉黑群
15. BUSINESS_BLOCKROOMMEMBER = 15 拉黑群成员
16. BUSINESS_LEAVEROOM  = 16 退群
17. BUSINESS_CANCELROOM  = 17 注销群
18. BUSINESS_BLOCKROSTERLIST = 18 拉取账号黑名单
19. BUSINESS_BLOCKROOMLIST  = 19 拉取账号拉黑群名单
20. BUSINESS_BLOCKROOMMEMBERLIST= 20 管理员拉取群黑名单

**toTid** 是Tid对象，为发送目标账号

**roomTid** 是Tid对象，为发送目标群账号

**udtype** 为开发者自定义值字段

**udshow**  为开发者自定义值字段

**extend** 为开发者自定义值字段 为字典类型字段

**extra** 为开发者自定义值字段 为字典类型字段

----------
## 客户端如何处理TimMessage信息
tim客户端为消息异步触发机制，执行客户端方法后，无需等待服务器返回数据，由服务推送结果到客户端，触发客户端监听程序执行既定流程。

tim的各个客户端都定义了MessageHandler方法或接口，通过实现MessageHandler(TimMessage)来处理服务传递过来的TimMessage对象。

以下是java 客户端实现处理TimMessage的demo程序片段：
    tc.MessageHandler((TimMessage tm) -> {
                if (tm.msType == 1) { //系统信息
                    Log.debug("this is system message >>", "body>>>", new String(tm.getBody()));
                } else if (tm.msType == 2) {  //用户到用户信息
                    Log.debug("this is user to user message");
                } else if (tm.msType == 3) {  //用户到群信息
                    Log.debug("this is room to user message");
                }
                switch (tm.odType) {
                    case 1: //常规消息
                        Log.debug("chat message >> from>>", tm.fromTid.node, " ,to>>", tm.toTid, ",body>>>", new String(tm.getBody()));
                        break;
                    case 2: //撤回消息
                        Log.debug("RevokeMessage>>>", tm.mid);
                        break;
                    case 3: //阅后即焚
                        Log.debug("BurnMessage>>>", tm.mid);
                        break;
                    case 4: //业务消息
                        Log.debug("business message>>>", tm);
                        break;
                    case 5: //流数据
                        Log.debug("stream message>>>", tm);
                        break;
                    default: //开发者自定义的消息
                        Log.debug("other message>>>", tm);
                }
            });


----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！