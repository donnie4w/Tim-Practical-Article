> tim通过TimPresence协议进行用户在线信息通信，主要定义数字属性字段，可以用于用户在线状态信息的通讯。
>[ 来源《tim实践系列——如何使用TimPresence自定义各种用户状态》](https://tlnet.top/article/22425172)

### TimPresence的字段：


| 表列 A | 数据类型 |是否必填 |作用 |值说明 |
| ----- | ----- | ----- | ----- | ----- |
|id | 	64位整型 | 否 |timPresence数据包id  | id值标识每一个timPrecence包，避免终端重复接收协议包 |  
| offline |布尔值  | 否 |是否离线  | 该值由系统通知，表示对方已经下线 |
|subStatus  |8位整型   | 否 |订阅状态  | 	该值由开发者自定义数值，用于订阅其他账号的状态 比如：1请求订阅 2已订阅并请求订阅3 不允许订阅 等等 该值非既定设定。 |
| fromTid |Tid对象	   | 否 |发送者	  |  |
| toTid |Tid对象	   | 否 | 目标对象	 |  |
| toList | 字符串数组	  |否  |多个目标对象账号  |  |
| show | 16位整型	  |否  |自定义显示类型  |开发者自定义显示类型，如chating,away,xa,dnd等  |
|status  | 字符串  |否  | 自定义显示类型 | 开发者自定义显示状态  如：我在打羽毛球 |
|extend    | 字典  |否  | 	扩展字段 | 字典类型，键值对，键字符串  值字符串 |
| extra   | 字典  |否  |扩展字段	  |	字典类型，键值对，键字符串  值字节数组  |

使用timPrecence时，需要对timPresence进行赋值，一些字段不需要客户端赋值的，包括 id，offline，fromTid 这些字段是由服务器赋值发送到客户端，所以客户端对这些字段赋值是无效的。

timPrecence所有值都不会影响tim的业务逻辑，所有字段都没有既定的设定值，除了id，offline，fromTid 是由服务器赋值发送之外的字段，tim服务器会将客户端发送的值原样发送给目标用户，所以，如何显示状态需要开发者自定义设定。根据目前主流im的状态功能，timPrecence定义了相应的字段。

## 字段说明：
**subStatus** 可用于向其他账号订阅状态，如 tim的java客户端处理subStatus的demo程序片段：

    final static byte SUBSTATUS_REQ = 1; //自定义订阅值
    final static byte SUBSTATUS_ACK = 2; //自定义回复订阅值
    //记录状态订阅者
    Map<String, Byte> submap = new ConcurrentHashMap<>();
    tc.BroadPresence(SUBSTATUS_REQ, (short) 0, "I am busy😄");//广播并订阅状态
    tc.PresenceHandler((TimPresence tp) -> {
                Log.debug(tp.fromTid.node, " presence");
                if (tp.subStatus > 0) {
                    if (tp.subStatus == SUBSTATUS_REQ) {  //对方账户订阅我的状态
                        tc.PresenceToUser(tp.fromTid.node, (short) 0, "", SUBSTATUS_ACK, null, null);//发送个人状态给对方
                    }
                    if ((tp.subStatus == SUBSTATUS_REQ) || (tp.subStatus == SUBSTATUS_ACK)) {
                       //记录订阅我的状态的好友账号
                        if (submap.containsKey(tp.fromTid.node)) {
                            submap.put(tp.fromTid.node, (byte) 0);
                        }
                    }
                }
            });

**toList** 当需要发送状态给多个账号时，可以将账号赋值给toList


----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！