> tim支持多种数据库 ：TLDB，Mysql，PostgreSQL ，SQL Server，Oracle，Oceanbase等。其中，默认数据库是tldb。使用tldb时，tim会在tldb建立用户关系表，群组关系表等业务表来支持tim的核心通讯接口和非核心的用户关系接口。如果不使用tldb时，可以通过配置sql来读取核心接口的数据来支持核心接口，但是tim不支持配置sql操作非通讯接口，如加好友，建群等。
订阅[《tim实践系列文章》](https://github.com/donnie4w/Tim-Practical-Article)随时查阅

使用tldb数据库时，默认支持tim内置账户系统，对于内置用户系统，tim是利用tim的核心接口实现了一套特定用户逻辑关系，实际上，这一套用户逻辑不一定适合所有业务的需求，但是，tim用户系统是对标目前流行的IM业务逻辑所设计开发的，适用于大部分的IM业务。如果tim的用户系统不能适配开发者的实际业务，开发者可以配置其他数据库，使用自己的用户系统。

**注意，外接数据库时，好友关系，群组关系，与tim通讯是没有直接关系的，无论是否好友，或是否群成员，判断依据是根据配置的SQL，查询后 反馈的发信权限是否为真来判断的。**

#### tim内置的业务接口如下：

**一. 用户关系接口**

1.   Roster()   获取花名册，即好友列表
2.   Addroster(String node, String msg)   申请加好友
3.   Rmroster(String node)    删除好友
4.   Blockroster(String node)   拉黑用户账号
5.   BlockRosterList()  用户黑名单

说明：tim的所有接口，除了注册和获取token接口外，其他接口都是异步返回数据。终端在tim客户端定义的对应handler捕获tim返回的数据并进行处理。

* Roster()是拉取好友账号的接口，返回账号的字符串数组
* Addroster(String node, String msg) 是向账号node发送加好友请求，msg为发送的消息，如我是XX等。如果对方同意加为好友，也需要同样调用Addroster加请求者为好友，即互相调用Addroster加为好友后，就是好友关系。
* Rmroster(String node)    是移除node账号的关系记录。除了移除好友关系，也同样适用于移除黑名单。如果拉黑某账号，也可以调用Rmroster移除该账号黑名单。
* Blockroster(String node)   拉黑用户账号node
* BlockRosterList()  用户黑名单列表，返回账号的字符串数组

**二. 群组接口**

1.   NewRoom(long gtype, String roomname)   创建群组
2.   AddRoom(String node, String msg)     申请入群
3.   PullInRoom(String roomNode, String userNode)   拉人入群
4.   RejectRoom(String roomNode, String userNode, String msg)   拒绝申请入群
5.   KickRoom(String roomNode, String userNode)  踢人出群
6.   LeaveRoom(String roomNode)  退群
7.   CancelRoom(String roomNode)  注销群
8.   BlockRoom(String roomNode)   用户拉黑群账户
9.   BlockRoomMember(String roomNode, String toNode)  群管理者拉黑群成员
10. BlockRoomList()  用户拉黑的群黑名单
11. BlockRoomMemberlist(String roomNode)  群管理者拉黑的群成员黑名单

* NewRoom(long gtype, String roomname) 需要传入两个参数 gtype与roomname，gtype有两个值gtype=1表示私有群，gtype=2表示公有群，私有群与公有群的区别在于用户加入群是否需要验证，类似微信群与QQ群 ，用户加入私有群需要管理员验证，加入公有群可以自由加入。
* AddRoom(String node, String msg)   申请加入群node，如果node是公开群，会直接入群，如果群为私有群，则需要群管理者拉入群。
* PullInRoom(String roomNode, String userNode)  群管理者可以拉人入群，如果有其他账号申请加入私有群，tim会通知群管理者，群管理者同意申请时，调用PullInRoom将申请账号拉入群，如果不同意，可以不理会或调用RejectRoom拒绝申请
* RejectRoom(String roomNode, String userNode, String msg)   群管理者拒绝入群申请并显示通知对方时拒绝原因时，可以调用RejectRoom方法，对方会受到没有通过申请的通知和原因。
* CancelRoom(String roomNode) 群的创建者在群成员全部退出后，可以调用CancelRoom将群注销
* BlockRoom(String roomNode)  用户可以通过拉黑群的方式，避免被群主或其他人拉入群
* BlockRoomList()  用户获取被其拉黑的群账号
* BlockRoomMember(String roomNode, String toNode)   群管理者可以拉黑用户账号toNode，该账号将不能再申请加入该群，如果时公开群，该账号也无法加入群。
* BlockRoomMemberlist(String roomNode)  管理员获取该群拉黑的成员账号列表

用户业务接口的调用与示例在[tim文档](https://tlnet.top/timdoc)中有详细说明：

![](/img/bVdaSVV)


----------

有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！