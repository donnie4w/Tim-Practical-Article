> TimMessage是tim最主要的数据结构，所以它的功能比较复杂，也比较多，主要体现在TimMessage时可以被持久化的，并且可以通过发送者或接收者发起相应的指令进行删除，TimMessage包含多个属性字段，多种数据类型属性，可以传递不同类型的数据。在《[tim实践系列——如何使用TimMessage自定义各种消息](https://tlnet.top/article/22425173)》中，详细介绍了TimMessage的数据结构，本文主要示例它的业务使用

## 系统消息：

系统消息时im重要的功能，主要由后台发起，展示给用户，tim提高了后台接口，通过https或http调用，发送数据给特定的一个或多个账号。在 tim使用文档 中 “tim后端管理接口—后台发送系统消息 ”  有使用说明
![](https://tlnet.top/f/1702539435_17516.jpg)

系统消息主要是使用TimMessage进行传递，调用后台管理接口,如：https://127.0.0.1:8001/timMessage  后，用户在messageHandler接口中会收到  msType=1的timMessage消息，如 [webtim中](https://github.com/donnie4w/webtim)，js的messageHandler实现：

        function parseMessage(tm, isPull) {
            // console.log(tm);
            //message消息
            if (tm.msType == 1) {  //系统消息
                console.log("this is system message >>", " body>>>", tm);
            } else if (tm.msType == 2) {  //用户到用户消息
                // console.log("this is user to user message");
            } else if (tm.msType == 3) {   //群到用户消息
                // console.log("this is room to user message");
            }

通过判断 tm.msType == 1确定 timMessage是系统消息，根据 开发者自定义的一系列字段值，对消息进行处理或展示



## 聊天消息：

tim客户端封装了多个聊天信息接口，主要是为了方便使用，实质都是调用同一个tim服务接口。如：

* MessageToUser(user, msg, udShow, udType)        向好友账号发送信息
* MessageToRoom(room, msg, udShow, udType)    向群账号发送信息
* MessageByPrivacy(user, room, msg, udShow, udType)   向群成员中非好友账号发送私信

调用以上接口，tim会向目标账号转发TimMessage数据，用户类似处理系统消息的方式，判断TimMessage的消息类型

* msType = 2   用户到用户的消息， 如果群账号roomTid不为空，则这是群成员发送的私信
* msType = 3   群到用户的消息
timMessage 通过msType判断消息的类型，通过 odType判断消息的指令，当odType=1是，表示这是一条普通聊天消息，如webtim中的普通消息判断及处理：

      switch (tm.odType) {  
      case 1: //常规消息
      let id = 0;
      if (tm.msType == 2) { //用户到用户消息
        id = chatid(tm.fromTid.node, tm.toTid.node);
      } else {  //群到用户消息
        id = chatid(tm.roomTid.node, tm.roomTid.node);
      }
      if (isEmpty(messageMap.get(id))) {
         initArrays(id);
      }
      if (tm.msType == 2) {
          //用户消息处理
          appendmsg(id, tm.fromTid.node, tm.toTid.node, false, unixnanoToDatatime(tm.timestamp), tm.dataString, 2, isPull);
      } else {
          //群消息处理
          appendmsg(id, tm.fromTid.node, tm.roomTid.node, true, unixnanoToDatatime(tm.timestamp), tm.dataString, 2, isPull);
      }
        break;
      case 2: //撤回消息
      console.log("revokeMessage>>>", tm.mid);
      break;
       .........
	  }

所以，tim是 通过msType与odType确定消息的来源，类型，及作用



## 撤回消息与消息阅后即焚

1. odType=1  常规消息，则，这是普通聊天信息
2. odType=2  消息撤回
3. odType=3  消息阅后即焚
当odType=2或odType=3时，表示该消息不是普通聊天消息，而是一个删除消息的指令，用户通过调用客户端接口来发起删除指令  

* RevokeMessage(mid, to, room, msg, udShow, udType)     消息撤回
* BurnMessage(mid, to, msg, udShow, udType)     消息阅后即焚

在RevokeMessage中 to 与room 两个参数是表示 撤回用户消息或是撤回群消息，二者只需要填写一个，另一个赋空值即可

RevokeMessage由消息的发送者调用，BurnMessage由消息的接收者调用，否则，服务器返回操作失败信息

终端调用RevokeMessage后，tim服务器会删除该消息，并同时向该消息的接收者发送一条TimMessage消息，指令类型odType=2，表示该消息被 发送者 撤回了，接收者收到后，应该同时删除本地储存相同mid的消息。

同理，终端调用BurnMessage后，tim服务器会删除该消息，并同时向该消息的发送者发送一条TimMessage消息，指令类型odType=3，表示该消息被阅后即焚，消息发送者收到后，应该同时删除本地储存相同mid的消息。

至此完成了 撤回消息与消息阅后即焚 的全部流程



## Tim的业务信息

#### 1. tim的接入外部业务

tim主要实现了通讯的核心功能，与数据通讯相关的接口或功能是tim的核心功能。与通讯没有执行关系的功能，如注册，加好友，建群等这些功能接口，是tim内置的业务接口，开发者可以选择用，也可以选择不用，选择不用时，可以配置非tldb的其他数据库，如mysql，Oracle等，通过配置sql支持tim核心接口的查询数据，保存数据等功能，类似注册，好友，群等功能可以自行实现，tim不会受影响。

如果自己实现好友与群功能，如何告知tim好友关系与群关系：通过配置SQL，如：

* tim.sql.user.auth  查询是否有用户到用户的发信权限，如sql：SELECT 1 FROM timfriend WHERE fromuid=? AND touid=? and status=1
返回1表示有权限发信。
 
* tim.sql.room.auth 查询是否有用户到群的发信权限，如sql：SELECT 1 FROM timgroupmember WHERE gid=? AND uid=? and `status`=1
返回1表示有权限发信。

类似这些SQL的配置与示例，在[TIM使用文档](https://tlnet.top/timdoc)中有详细说明
![](https://tlnet.top/f/1702542971_18963.jpg)
#### 2. tim的内置业务

如果使用tim默认数据库tldb，则tim启动时，会在tldb中建立相应的数据表来支持tim内置实现的一套用户关系功能，如注册，好友关系，群操作等功能，可以使用tim提供的相关接口

在[TIM使用文档](https://tlnet.top/timdoc)中有说明

![](https://tlnet.top/f/1702543359_15732.jpg)

客户端调用相关接口后，如加好友，入群等，tim会相向相关账号推送TimMessage消息，如webtim中的处理：

    switch (tm.bnType) {
                
    case STAT.BUSINESS_REMOVEROSTER: 
        .... //被解除除好友关系
    case STAT.BUSINESS_ADDROSTER:
        ....  //加好友请求
    case STAT.BUSINESS_FRIEND:
        ....  // 成为好友
    case STAT.BUSINESS_ADDROOM:
        .... // 入群申请
    case STAT.BUSINESS_PASSROOM:
       tc.StreamToRoom(tm.roomTid.node, null, 1, 0);
       tc.UserRoom();
       alert("入群申请已经通过:" + tm.roomTid.node);
       break
    case STAT.BUSINESS_PULLROOM:
         ....  //被拉入群
    case STAT.BUSINESS_KICKROOM:
       console.log("被移除出群>>>", tm);
       break
      ......
    
    }

**odType=4时，表示这是tim的内置业务操作指令**

tim的内置业务操作通知，通过  bnType 的值来判断 操作类型，在《[tim实践系列——如何使用TimMessage自定义各种消息](https://tlnet.top/article/22425173)》中有说明。

## 基于TimMessage的流数据

tim中定义了两种流数据，一种是基于timStream的流数据，另外是基于timMessage的流数据，timStream主要用户数据量较大的流数据分发，类似直播间这样的业务类型，它需要建立虚拟房间来支持。基于timMessage的流数据 实际上传送的也是timMessage对象，但是区别在于，timMessage流数据不会被持久化，没有离线消息，只能发送到在线用户，用户处于离线状态时，该消息就直接抛弃

**odType=5是，表示该TimMessage是流数据对象**，客户端接口为：

* StreamToUser(to, msg, udShow, udType)   发送到用户
* StreamToRoom(room, msg, udShow, udType)  发送到群
如， 音视频电话，多人音视频会议等功能，可以使用 timStream来实现，也可以使用 timMessage流数据来实现。

使用timMessage流数据相对简单，发送流数据与发送普通消息只是接口不同，方式一样，数据在tim中流程也一样。






----------


有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！
