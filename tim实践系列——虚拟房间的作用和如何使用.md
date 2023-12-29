> 流数据传输是互联网应用中一个十分重要的基础功能。而流数据的支持是tim中最重要的基础功能之一，tim的流数据实现同样支持1:1,1:n,n:n等基础通讯模式，这些模式几乎涵盖所有流数据的应用场景。
> [《TIM即时通讯引擎》](https://tlnet.top/tim)
> 订阅[《tim实践系列》](https://github.com/donnie4w/Tim-Practical-Article)文章，了解如何使用tim

### tim流数据在实际中的应用

#### 1. 如在webtim：https://tim.tlnet.top 中实现音视频直播功能：

![图片](https://tlnet.top/f/1703839711_5615.jpg)
 
![图片](https://tlnet.top/f/1703839727_20027.jpg)
 
![图片](https://tlnet.top/f/1703839757_442.jpg)

#### 2. 在webtim：https://tim.tlnet.top 中实现实时文件传输 功能：

![图片](https://tlnet.top/f/1703840191_18609.jpg)
 
![图片](https://tlnet.top/f/1703840201_31151.jpg)
 
![图片](https://tlnet.top/f/1703840210_20697.jpg)

以上是两个涉及流实时传输的应用场景

----------

### 在tim中有两个流数据传输方式：

* 基于[TimMessage](https://tlnet.top/article/22425173)
* 基于TimStream

#### 说明：

1. 基于TimMessage的流实现与普通消息实现在流程与用法上没有区别，但是它属于现场数据，不会被持久化。而且需要做发送与接收的权限验证，流传输基于两个终端的账号。由于流程上的复杂性，它适用于流传输压力较小的场景。
2. 基于TimStream的流实现是tim最主要的流实现，它基于虚拟房间账号传输，与用户账号无关，没有复杂的流程，几乎达到端对端的效果，而且是一般是1：n的传输模式。通过tim去中心化的集群模式做水平扩展后，可以支持海量的流分发。适用与像直播，多人音视频会议，线上视频远程办公，VR实时数据等业务场景。


### 本章主要讲解tim虚拟房间的用法：

虚拟房间的接口有：

1. 创建虚拟房间
2. 注销虚拟房间
3. 虚拟房间加权
4. 虚拟房间除权订阅
5. 虚拟房间取消订阅
6. 虚拟房间发送流数据

    以下是tim的java客户端atim的虚拟房间demo程序片段： tc.VirtualroomRegister(); //注册虚拟房间
    tc.VirtualroomRemove("NGhMpCbk2wQ"); //移除虚拟房间，该方法一般无需调用，虚拟房间无数据流后会自动注销
    tc.VirtualroomAddAuth("NGhMpCbk2wQ","100001"); //虚拟房间创建人给其他账号添加向虚拟房间推送流数据的权限
    tc.VirtualroomDelAuth("NGhMpCbk2wQ","100001"); //移除推送流数据的权限
    tc.VirtualroomSub("NGhMpCbk2wQ"); //订阅虚拟房间
    tc.VirtualroomSubCancel("NGhMpCbk2wQ"); //取消订阅虚拟房间
    tc.PushStream("NGhMpCbk2wQ",new byte[]{1,2,3,4}, (byte) 0); //推送流数据

### 接口说明：

#### 基于虚拟房间的推流，最终实现的是向tim推流，并进行流的分发
![图片](https://tlnet.top/f/1703752363_1783.jpg)

#### 在集群环境中：

![图片](https://tlnet.top/f/1703752883_30091.jpg)

tim自身实现的集群算法，自动形成这样的分布式架构，不需要人为调整，也无法人为干预。
中继服务器只是一个方便理解的称呼，实际上也是tim集群中一个节点。任何一个tim节点都可能成为某个流数据的中继器，这需要tim通过算法计算等到。而且在流分发中，一个流的中继服务器会派生流数据第二备用，第三备用，第四备用...等等的多个备用tim节点，当该中继器断开或其他节点无法连接它时，可以连接到它的后续备用节点上，中继器与集群断开时，第二备用节点变成这个流数据的中继服务器，它会继续派生出多个备用节点。而连接备用节点或派生备用节点等行为都是通过算法自动完成的，无法人为部署。

----------

### 推流具体实现：

注册虚拟房间，获取房间号：
1. 客户端调用 tc.VirtualroomRegister(); 获取虚拟房间号，如：NGhMpCbk2wQ此时，注册者可以向虚拟房间 推送流数据，其他账号则无权限向该虚拟房间推送流数据。
2. 其他账号订阅该虚拟房间的流数据：tc.VirtualroomSub("NGhMpCbk2wQ")
3. 向NGhMpCbk2wQ推送流数据：tc.PushStream("NGhMpCbk2wQ",new byte[]{1,2,3,4}, (byte) 0)

**这样，订阅者在终端可以获取流数据。**

订阅者可以取消订阅  tc.VirtualroomSubCancel("NGhMpCbk2wQ");

当虚拟房间号 一段时间没有流数据发生时，会被系统自动销毁掉。或者建立者通过调用tc.VirtualroomRemove("NGhMpCbk2wQ");注销掉虚拟房间。

**说明：虚拟房间号的产生是随机不重复的。**

虚拟房间的各个方法调用与返回，在[tim开发使用文档](https://tlnet.top/timdoc) 中有详细讲述
![图片](https://tlnet.top/f/1703754580_14671.jpg)

----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！
