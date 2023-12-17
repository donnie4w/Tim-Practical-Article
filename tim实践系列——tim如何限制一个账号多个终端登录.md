> 部分im应用允许同一个账号同时多终端登录，如微信，QQ等应用，允许同账号在移动端与PC端同时登录。tim服务器支持通过配置，灵活设置并扩展该功能
> 来源[《tim实践系列》](https://github.com/donnie4w/Tim-Practical-Article)

### Tim登录
tim客户端 通过TimAuth协议对象登录tim服务器，TimAuth的字段说明：

| 字段名 | 类型 |作用 |
| ----- | ----- |----- |
| name | 字符串 |登录用户名 |
| pwd| 字符串 | 登录账号|
|domain    | 字符串 |域名  |
| resource |字符串 | 资源 |
| termtyp |整型 | 终端类型 |
|token  | 字符串|登录token  |
| extend |字典|扩展字段  |

### 登录方式：

* 用户名密码    通过name与pwd进行登录验证
* token            通过token登录

#### 其他属性字段是选填字段

* domain     域名，用以分隔不同区域的账号，不同domain的账号无法通讯。但是外接的用户系统不受domain限制。
* resource   用来传递终端设备的信息，如  huawei50
* termtyp    用来标识终端类型，tim服务器主要通过该字段限制同账号同类型终端
* extend    扩展字段

## termtyp 说明

termtyp  终端类型 标识登录终端的类型，数据类型是8位整型 ，该字段由开发者自定义数据值，tim服务器通过该属性，判断终端是否属于同一种类型。

开发者可以定义termtyp 区分PC端与移动端，也可以细分各种类型的终端，需要根据业务需求而定。

    以下是示例：
    比如定义termtyp，只区分pc端与移动端
    termtyp=1   移动客户端
    termtyp=2   桌面客户端
    
    再比如定义termtyp，将终端分为4类
    termtyp=1   Android客户端
    termtyp=2   ios客户端
    termtyp=3   web客户端
    termtyp=4   电脑桌面客户端
    
    或者比如 定义termtyp，将终端分为8类
    termtyp=1   Android-华为
    termtyp=2   Android-小米
    termtyp=3   Android-oppo
    termtyp=4   Android-vivo
    termtyp=5   Android-其他
    termtyp=6   ios客户端
    termtyp=7   web客户端
    termtyp=8   电脑桌面客户端


termtyp的值是开发者自定义的。tim对终端类型的判断是根据termtyp字段。


----------

**接下来看一下tim限制账号登录的配置文件**

tim的配置文件有两个字段限制登录账号：

tim.json

     {
     ...
    "device.limit":1,
    "devicetype.limit":1
     ....
     }

* device.limit = 1  表示限制tim 一个账号同时只能一处登录。 tim根据device.limit的值限制同一个账号登录终端数，不分终端的类型。
* devicetype.limit = 1  表示限制tim 同一类型的终端情况下，同一账号同时只能允许一个终端登录，终端类型根据 timAuth协议中的termtyp值来判断。tim会统计一个账号同一个termtyp值的终端数，根据devicetype.limit限制终端数
* device.limit的缺省默认值是1，devicetype.limit的缺省默认值是1


以下是示例：
比如配置场景  一：
 
    {
     "device.limit":2,
     "devicetype.limit":1
     }

    开发者设置：
    
    termtyp=1   移动客户端
    termtyp=2   桌面客户端
    
    表示：同一个账号最多只能两个终端同时登录，即移动客户端与桌面客户端同时能登录同一个账号，类似QQ
    
    java demo程序
    final static byte MOBILE = 1;  //移动终端
    final static byte DESKTOP = 2;  //桌面终端
    tc.Login("jerry", "123", "github.com", "ios-20-plus", MOBILE, null);
    //通过调用Login方法，设置termtyp为MOBILE，即 termtyp=1 . 
    //tim服务器将统计termtyp=1的终端数，根据配置"devicetype.limit"进行判断同一账号是否超过登录数

比如配置场景 二：

     {
    "device.limit":4,
    "devicetype.limit":1
     }

    开发者设置：    
    termtyp=1   Android客户端
    termtyp=2   ios客户端
    termtyp=3   web客户端
    termtyp=4   电脑桌面客户端

    表示：同一个账号最多能有4个终端同时登录，即Android客户端，ios客户端，web客户端，桌面客户端 同时能登录同一个账号。
    demo程序与上同.

同一账号后登录被tim限制无法登录时，会收到反馈信息：

错误码：4111, 错误信息 ："over entry"

此时，客户端应该调用Logout()方法退出登录关闭连接，或者开发者可以自定义消息，通过调用tim后台接口，发送系统通知给已经登录的个人账号，要求其退出登录，然后可以再次尝试登录。


----------


有任何问题或建议请Email：donnie4w@gmail.com或 https://tlnet.top/contact  发信给我，谢谢！

