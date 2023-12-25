> 消息撤回与阅后即焚是tim的两个核心接口，tim的数据传输核心接口有 消息接口，消息撤回，阅后即焚，流数据接口，另外几个核心接口在《tim使用文档》有详细说明
> 订阅《[tim实践系列文章](https://github.com/donnie4w/Tim-Practical-Article)》随时查阅

![](https://tlnet.top/f/1703041489_22809.jpg)

常规消息接口也就是通常定义的聊天消息

**注意：消息撤回接口必须由 消息发起方调用才有效，否则tim服务器返回权限错误。消息撤回可用于 用户到用户的消息撤回，也可以用于用户到群消息的撤回.**

	java客户端demo 程序示例：

    tc.RevokeMessage(123L,"10001", null,  null, (short) 0, (short) 0);  //用户到用户的消息撤回
    //撤销 消息id 为123 的消息，
    //10001 为消息的接收方
    
    tc.RevokeMessage(123L,null, "100000001",  null, (short) 0, (short) 0);   //用户到群的消息撤回
    //撤销 消息id 为123 的群消息
    //100000001 为群账户


----------

**注意，阅后即焚接口必须由 消息接收方调用才有效，阅后即焚可用于 用户到用户的消息阅后即焚，群消息没有阅后即焚接口**

    tc.BurnMessage(123L, "10001",   null, (short) 0, (short) 0);
    //对消息id 为123 的消息阅后即焚，
    //10001 为消息的发起方

![](https://tlnet.top/f/1703041944_16943.jpg)

![](https://tlnet.top/f/1703041956_30385.jpg)

**调用 消息撤回或消息阅后即焚接口后，tim服务器会返回TiimAck给调用者，并发送TimMssage给消息的相关账号，终端通过ackHanlder与messageHandler处理数据结果，如webtim中处理：**

ackHandler:

      ......
      case STAT.TIMREVOKEMESSAGE:
         console.log("撤回消息返回", ta)
          if (ta.ok) {
              let tonode = ta.n;
              let t = ta.t;
              let mid = ta.ackInt;
			  let cid = 0;
              if (t == 2) {
                 cid = chatid(myAccount, tonode);
                 let m = midMap.get(cid + "_" + mid);
                 if (!isEmpty(m)) {
                     m.isRevoke = true;
                 }
              } else if (t == 3) {
                 cid = chatid(tonode, tonode);
                 let m = midMap.get(cid + "_" + mid);
                 if (!isEmpty(m)) {
                     m.isRevoke = true;
                 }
              }
             if (document.getElementById("accountId").value == tonode) {
                 showchat(cid)
              }
          }
          break;
     case STAT.TIMBURNMESSAGE:
         console.log("阅后即焚返回", ta)
     ......


----------

 messageHandler:

    ......
    case 2: //撤回消息
        console.log("revokeMessage>>>", tm);
        let cid = 0;
        let node = "";
        if (tm.msType == 2) {
            node = tm.fromTid.node
            cid = chatid(node, tm.toTid.node);
        } else if (tm.msType == 3) {
            node = tm.roomTid.node;
            cid = chatid(node, node);
        }
        let m = midMap.get(cid + "_" + tm.mid);
        if (!isEmpty(m)) {
            m.isRevoke = true;
        }
        if (document.getElementById("accountId").value == node) {
            showchat(cid)
        }
        break;
	.......

撤回消息与阅后即焚接口是没有时间限制的。在实际业务中，可以根据实际需要，加上撤回的时间限制。而阅后即焚，可以通过一般信息阅读时间，如8秒或10秒等，在消息显示后，调用阅后即焚接口。


----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！