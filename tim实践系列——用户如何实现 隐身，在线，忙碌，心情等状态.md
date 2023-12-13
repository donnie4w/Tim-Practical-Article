> tim通过TimPresence传递在线信息，该对象不会被持久化，只有在线用户可以收到。一般可以用于用户在线状态的传递，《[tim实践系列——如何使用TimPresence自定义各种用户状态》](https://tlnet.top/article/22425172) 介绍了timPresence的各个属性作用，本文介绍一般用户状态使用TimPresence的示例

部分IM软件会有显示用户在线状态的功能，如QQ，Skype等

tim本身没有用户状态的说法，也就是说TimPresence不一定非得用于定义用户状态，也可以用于发送用户信息，但是TimPresence对象不会被持久化，也没有离线信息，只有在线用户才能收到，所以它的特征比较适合用于传递用户在线状态。而且用户掉线时，tim的内部业务逻辑也是发送一条TimPresence信息给好友用户，其属性offline=true。

TimPresence的各个属性字段都由开发者自定义，使用timPrecence没有既定字段值，可以自由设置，自由解析，所以，实现 隐身，在线，忙碌，心情等状态，需要开发者定义TimPresence中的某一个字段后自行解析，tim是没有内置的默认值的。

如：定义TimPresence中show的字段：

java示例

    final static short SHOW_CHAT = 1;   //聊天中
    final static short SHOW_AWAY = 2;   //离线
    final static short SHOW_XA = 3;     //走开
    final static short SHOW_DND = 4;    //勿扰

这样赋值给show后，对方收到show值就知道发送者的状态

用户上线后，可以调用客户端接口  **BroadPresence** 来广播自己的状态，并同时订阅其他好友的状态:

如：

**tc.BroadPresence(subStatus,show ,status);**

这个接口定义了3个字段：

* 第一个对应TimPresence的字段subStatus   8位整型
* 第二个对应TimPresence的字段show          16位整型
* 第三个对应TimPresence的字段status            字符串
这样就足以传递用户的状态及行为。

如在[webtim项目](https://tim.tlnet.top)中
![](https://tlnet.top/f/1702439298_18608.jpg)
用户上线时，调用   tc.BroadPresence(1,0,“😄”);

其他用户收到数据：

subStatus=1表示他要订阅我的状态，所以我记录他的状态，并回复我的状态给他 tc.PresenceToUser("Ximkeig13y", 0, "😄", 1, null, null))
show=0 这里没有定义它作用
status=😄 显示在用户状态上
这样完成了webtim的用户状态信息传递。

PresenceToUser 发送特定账户
PresenceToList  发送给多个特定账户
说明：webtim源码：https://github.com/donnie4w/webtim



以下时java使用TimPresence的示例：

    final static byte SUBSTATUS_REQ = 1;  //订阅状态
    final static byte SUBSTATUS_ACK = 2;  //回复订阅状态
    final static short SHOW_CHAT = 1;   //聊天中
    final static short SHOW_AWAY = 2;   //离线
    final static short SHOW_XA = 3;     //走开
    final static short SHOW_DND = 4;    //勿扰
    
    final static String STATUS_EMOTION_HAPPY = "I'm happy";   
    final static String STATUS_EMOTION_SAD = "I'm down";   
    final static String STATUS_EMOTION_SUNSHINE = "☀️☀️☀️";   

//向账号10001发送字节的状态
tc.PresenceToUser("10001",SHOW_CHAT,STATUS_EMOTION_SUNSHINE,SUBSTATUS_REQ, null, null); 

//向所有好友广播个人状态并订阅对方状态
tc.BroadPresence(SUBSTATUS_REQ,SHOW_CHAT,STATUS_EMOTION_SUNSHINE);
注意： SHOW_AWAY = 2;    这里订阅的离线与offline为true的离线是不同的。

SHOW_AWAY = 2;   是开发者自定义的离线，可以理解为隐身，账号实际还是在线的，可以对标QQ的隐身功能

timPrecence的offline为true是，是tim服务器发送的，表示账号与服务器已经断开连接了。

当用户掉线时，tim服务器会给她的好友发送一条TimPresence信息，改信息通过实现 客户端 presenceHandler来处理：

如 webtim：



        //状态消息
    tc.presenceHandler = function (data) {
        let tp = new TimPresence()
        tp = JSON.parse(data);
        console.log(tp);
        if (!isEmpty(tp.subStatus)) {
            if (tp.subStatus == 1) {
                tc.PresenceToUser(tp.fromTid.node, 0, "😄", 2, null, null);
            }
            if (tp.subStatus == 1 || tp.subStatus == 2) {
                if (isEmpty(submap[tp.fromTid.node])) {
                    submap[tp.fromTid.node] = 0;
                }
            }
        }
        if (tp.offline && (tp.fromTid.node == myAccount)) {
            tc.BroadPresence(1, 0, "haha");
        }
        if (tp.offline) {
            friendstatus(tp.fromTid.node, false, "", false);
        } else {
            friendstatus(tp.fromTid.node, true, tp.status, false);
        }
        console.log("presence>>>", tp);
    };

其中：

     if (tp.offline) {
         friendstatus(tp.fromTid.node, false, "", false);  //设置用户为离线状态
     }

offline=true是服务器通过捕获 用户掉线信息后，发送给其好友的信息

该信息也可以通过配置tim启动参数，禁止发送，该参数为  **presence.offline.block**

如tim.json

    {    
     ......    
    "presence.offline.block":true    
     ......    
    }    


**presence.offline.block**默认为false，赋值为true后，tim将不再广播 用户离线消息



----------


有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！