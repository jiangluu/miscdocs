## 基本介绍
本项目是某棋牌游戏的平台部分。

## 思路
把房间之外的接口都平台化。平台化的含义主要是：
- 不管具体游戏是麻将或斗地主或其他都用同一套
- 分布式设计，故可以扩展到支持百万玩家

分界线在 join_room。join_room及之后的消息属于房间之内，还是发送到原先的java后端（使用原先的websocket协议）。

其他的消息（包括登录、商城、战绩，创建房间等）都发往平台，并且使用HTTP协议。

## 接口通用规则
- 接口是HTTP或HTTPS协议。原则上都是POST请求
- 除了登录接口本身，其他接口都需要登录后才能使用
- 参数都封装成一个json串，在请求的Body里（而不是在uri里）
- 返回标准HTTP Response

## 接口返回值
返回值是一个json字符串，有三种不同的情况：
- { "error": 错误号 }。表示请求由于某种原因失败 （错误号含义请看文档最后部分）
- { "OK":1 }。 表示请求成功
- 不属于以上两种的。这说明此请求是获取数据类型的，并且把数据以json的形式传递给前端。

注意以上所说的返回都是在HTTP的上一层。如果前端请求的url错误，HTTP本身会返回404之类，根本不会进入我们的系统。

## 服务地址
目前开发阶段地址为 http://118.25.8.189:38080/ 。客户端应把此地址写成可配置的。

此地址拼上具体接口就是完整的url， 如ping接口的完整url就是 http://118.25.8.189:38080/api/v1/ping 。

## 如何调试接口
在客户端代码中写完逻辑后，靠实际运行游戏来调用后端接口比较繁琐，建议使用工具直接向后端发送请求，来验证后端接口的正确性。

较常用的工具是curl（windows不自带）；建议可使用chrome浏览器安装插件 Restlet Client - REST API Testing， 使用非常方便。

## 已调通接口
为了醒目，这里先列出后端已经调通的接口，及所使用参数、和返回值。也可以把它们当作例子看待。

|接口|输入|返回|
| :-- | :--  | :-- |
| /api/v1/ping |无|Your IP is: xxx|
| /api/v1/check_version | {"v":1} |OK|
| /api/v1/login_by_openID |{"openID":"Og26tzlEbNpdhqru"}|{"ver":1,"isguest":1,"ban_score":0,"ctime":1517038051,"RMB":0,"ban_gold":10000,"usn":100000,"gold":0,"nick":"default user","ban_diamond":10,"diamond":0,"score":0,"credit":0,"vip_level":1}|
| /api/v1/login_guest | {"a":"ASDF"}|{"openID":"Og26tzlEbNpdhqru"}|
| /api/v1/create_room |{"openID":"Og26tzlEbNpdhqru","N":4, "room_type":"SRGP","room_param":"ZR0;1FEN;8JU;FD8;FZPAY"}|{"room_key":"751958"}|
| /api/v1/get_user_room | {"openID":"Og26tzlEbNpdhqru"}|{"room_type":"SRGP","secure":"pKqUwVzyVbDhjaOF","openID":"mRq2gbvm41iChLSY","joined":["1"],"N":4,"url":"ws:\/\/8.8.8.8:88","owner":"1","key":"987108","room_param":"ZR0;1FEN;8JU;FD8;FZPAY"}|
| /api/v1/join_room |{"openID":"Og26tzlEbNpdhqru","key":"751958"}|{"room_type":"SRGP","secure":"pKqUwVzyVbDhjaOF","openID":"mRq2gbvm41iChLSY","joined":["1"],"N":4,"url":"ws:\/\/8.8.8.8:88","owner":"1","key":"987108","room_param":"ZR0;1FEN;8JU;FD8;FZPAY"}|
| /api/v1/quit_room | {"openID":"Og26tzlEbNpdhqru"} |OK|
| /api/v1/check_mail | {"openID":"mRq2gbvm41iChLSY"} |{"mails":[{"title":"标题","id":"d6YhevOsr9SMDT2P","ban_diamond":1000,"ban_score":1000,"ctime":1519272370,"context":"正文见过京东大佬在北京的豪宅，见过刘强东宿迁老家的别墅吗？","ban_gold":1000},{"title":"标题","id":"FdBTp4ehrQe8kj1r","ban_diamond":1000,"ban_score":1000,"ctime":1519272370,"context":"正文见过京东大佬在北京的豪宅，见过刘强东宿迁老家的别墅吗？","ban_gold":1000}],"refresh":{"gold":1000},"unread_mail":true}|
| /api/v1/pay_begin | {"openID":"39gowN8wCuNfuh5V", "productID":"cheshi2","android_or_iOS":"1","pay_type":"0" } |{"status":0, ,"type":1,"code":3,"pay_url":"https://xxxxxx" }|
| /api/roomserv/fetch_user_by_usn | {"app_secure":"cf7jvlKrhvCIqrfJM6cp", "usn":1} |{"ver":1,"isguest":1,"ban_score":0,"ctime":1517038051,"RMB":0,"ban_gold":10000,"usn":100000,"gold":0,"nick":"default user","ban_diamond":10,"diamond":0,"score":0,"credit":0,"vip_level":1}|
| /api/roomserv/fetch_room | {"app_secure":"cf7jvlKrhvCIqrfJM6cp", "key":"751958"}|{"joined":[{"secure":"fZAEwx81ynkGI3Hb","usn":"1"}],"room_type":"SRGP","openID":"mRq2gbvm41iChLSY","N":4,"url":"ws:\/\/8.8.8.8:88","key":"987108","owner":"1","room_param":"ZR0;1FEN;8JU;FD8;FZPAY"}|


## 具体接口说明
##### /api/v1/ping
健康检查（业务上不需要，只是程序上用，或者可用于测试网速等）

### 客户端用
下面所列这些API是为客户端访问准备的。此外还有为实现具体游戏玩法的游戏服务器准备的API，请查看 游戏服务用 一节。
##### /api/v1/check_version
检查客户端版本是否够新。

参数：v:客户端版本号

返回OK表示成功。

##### /api/v1/login_by_openID
使用openID登录。严格来说，只提供这一种登录方式。其他方式（手机号、微信等）是用于换取openID，然后还是要调用本接口登录。

若openID过期，则让用户再使用手机号、微信等方式换取新的openID。

参数：openID

返回：包含所有用户数据的json串，例子： {"ver":1,"isguest":1,"ban_score":0,"ctime":1517038051,"RMB":0,"ban_gold":10000,"usn":100000,"gold":0,"nick":"default user","ban_diamond":10,"diamond":0,"score":0,"credit":0,"vip_level":1}

##### /api/v1/logout
下线。当客户端关闭时，尽可能调用这个。有利于做在线时长统计等

参数：openID

##### /api/v1/login_guest
游客账号登录。  参数：a:唯一串号。此串号由客户端生产，尽量保证唯一性。串号应该在客户端本地保存。

返回openID，前端应在本地保存。

##### /api/v1/login_phone
手机号登录。

参数：一个json字符串。此json中必须包含以下字段：
> - phone：手机号
> - zone：手机区号
> - code：手机验证码
> - mob_appkey：MOB给此APP的key
> - passwd：密码（的MD5）  
> 可以包含更多的字段，后端会保存下来。
例子：
```json
  {
    "phone": 13112345678,
    "zone": "+86",
    "code": "6379",
    "mob_appkey": "ABCD",
    "passwd": "AJKFJRE6JHNGH9"
}
```
其他请求的参数也是封装成类似的json字符串，不再一一举例。

返回openID，前端应在本地保存。

##### /api/v1/login_wx
使用微信登录。 

参数：客户端拉起微信授权后，微信返回的json字符串。

返回openID，前端应在本地保存。

##### /api/v1/login_qq
使用QQ登录。 

参数：客户端拉起QQ授权后，QQ返回的json字符串。

返回openID，前端应在本地保存。


##### /api/v1/bind_phone
绑定手机号。注意必须在登录后才能调用。下同。

参数：一个json字符串。此json中必须包含以下字段：
> - openID：openID 
> - phone：手机号
> - zone：手机区号
> - code：手机验证码
> - mob_appkey：MOB给此APP的key
> - passwd：密码（的MD5）  

返回OK表示成功。

##### /api/v1/bind_wx
绑定微信。

参数：客户端拉起微信授权后，微信返回的json字符串，在此json中添加一个openID字段

返回OK表示成功。

##### /api/v1/bind_qq
绑定QQ。

参数：客户端拉起微信授权后，微信返回的json字符串，在此json中添加一个openID字段

返回OK表示成功。

##### /api/v1/pull_board
拉取公告（必须在登录后）。

参数：openID:openID

返回json串，包含当前所有公告

##### /api/v1/check_mail
检查邮件，同时更新自己的钻石、金币等货币。

参数：openID:openID   l,m: 拉取从第l封到第m封的邮件。从0开始。可以都省略，后端默认拉取前5封邮件。

返回json串。最简单的结果是 {"unread_mail":false} ，表示没有未读邮件（但可以有已读邮件）。客户端可以根据 unread_mail 字段显示红点。如果有邮件（不管已读未读），会有 mails字段，是一个数组。如果有货币改变，会有 refresh字段，包含货币的最新值。

##### /api/v1/pay_begin
开始付费充值流程。整个充值流程根据第三方支付的文档归纳如下图：

![image](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/63320844712d2cda0dfbcc373f96798a46ac11d468050a6fede4e59aaf549924300881fa66d3ba79a8077c12e0d4f21d?pictype=scale&from=30113&version=2.0.0.2&fname=OIROEE6%5D%7B0S%2463E_IVN%28DIJ.png&size=1024)

参数：openID：openID  productID：要买东西的id  android_or_iOS：字符串1-客户端是安卓，字符串2-是iOS  pay_type：字符串0-微信支付，字符串1-支付宝支付

返回：预订单，json格式。内容就是第三方支付给平台的，平台不加修改。其中最重要的是 status字段，如为0表示下单成功，客户端可调起支付SDK；不为0则是各种错误。

附加说明：由于平台与客户端是短连接，支付后，客户端需要向平台拉取充值结果。通过 check_mail 接口拉取。


##### /api/v1/get_user_room
查询自己当前所在房间

参数：openID

返回：一个json串。如果当前不在房间里，json串里没有内容；反之则包含房间信息

##### /api/v1/quit_room
离开当前自己所在房间。不管自己当前在不在房间里，此请求都可以调，并返回成功。

参数：openID

返回OK表示成功。

##### /api/v1/create_room
新建房间。

参数： openID:通用  N:房间人数   room_type:此房间内具体游戏类型  room_param:此房间内具体游戏参数
例子： { "N":4, "room_type":"SRGP","room_param":"ZR0;1FEN;8JU;FD8;FZPAY","openID":"ABCD1234" } 

详细说明：作为平台，需要支持多个游戏，且以后很可能随着业务开展越来越多；平台对于 room_param 参数表示什么含义是完全不知道的，只是代为存贮下来；平台对于 room_type 参数的含义也所知十分有限，平台只负责找到能处理此种游戏类型的游戏服务器而已。支持的游戏类型需要与客户端约定。

返回：json串 {"room_key":"6位数字"}

##### /api/v1/join_room
请求进入房间。即使是创建房间的人，如果想玩游戏也需要调用此API进入房间。因为我的理解中，开房间的人不一定进去玩（如果此理解不对则修改这里）

参数： openID:通用  key：房间钥匙

返回一个json串，包含房间信息(当初客户端创建房间时指定的)，游戏服务器url，可以据此去连接负责此房间的具体游戏服务器。


### 游戏服务用
游戏服务用API有一个通用参数 app_secure， 每个请求都需要包含这个参数， 值目前是 cf7jvlKrhvCIqrfJM6cp 。
##### /api/roomserv/fetch_user_by_usn
根据usn获取用户信息。

参数： app_secure：通用； usn：用户id（客户端连接游戏服务器以后应该上报的）。

返回：json串，包含此用户所有信息。

##### /api/roomserv/fetch_room
根据房间key获取房间信息。

参数： app_secure：通用； key：房间key

返回：json串，包含房间信息（创建房间时设定了哪些信息，这里就会得到哪些）

##### /api/roomserv/push_room_round_stat
游戏服务器推送游戏简报给平台。

参数： app_secure：通用； key：房间key   round：第几局（从1开始）   stat：9-此局结算 1-此局开始打   
over：为非0值表示房间规定的局数都打完了    report：简报内容，当stat是结算时需要有这个字段，具体内容需要与游戏服务器商议

返回：OK表示推送成功。（因为这个数据很重要，如不成功游戏服务器应重试至少5次）


### 错误码
|错误码|表示含义|
| :----------------------------: | :-----------------------------------: |
|215|令牌(openID)不存在或失效|
|220|账号被冻结|
|222|客户端版本不支持|
|224|app_secure无效|
|301|必须提供密码|
|302|密码不符|
|303|必须提供手机号码|
|304|手机号码不是有效的格式（11位数字）|
|306|手机号码已被占用|
|307|手机验证失败|
|308|不合法的唯一串号（游客登录时）|
|309|微信验证失败|
|310|QQ验证失败|
|311|身份证号不是有效的格式|
|320|不合法的房间类型（创建房间时）|
|321|查无此房|
|322|你已经在一个房间里，不能再加入房间(离开房间后可以)|
|323|房间已经满了(这个是否应该由平台判断？)|
|324|查无此人|
|325|未找到制定id的邮件|
|326|已经实名过|
|329|俱乐部错误|
|330|商品id不存在|
|331|此客户端不支持的游戏类型|
|332|大奖赛此时不开放|
|333|您不是高级推广员|
|335|您在俱乐部中的权限不够|
|334|俱乐部不允许申请|
|335|没有此回放录像|
|340|货币不够|
|380,381|其他内部错误|
|403|必要参数不存在|
