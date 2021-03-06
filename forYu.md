## 后台总体设计思路
- 后台与游戏核心应是隔离的。后台不管是没有上线，或者是在做繁重的数据统计工作，都不应对玩家的游戏体验有丝毫的影响
- 后台不但包括可视化网页，更重要的是包括整个游戏运营数据；游戏核心使用的数据存储方案是基于NoSQL的，应该不能满足后台的需求，后台需要自己存储运营数据

基于以上两条认识，最粗略的后台框架是：
- 后台暴露一个消息队列服务，游戏核心在游戏运行的过程中，会不断的push事件到此队列（事件不但包括操作也包括更新后的玩家数据）
- 后台业务进程通过消耗这个消息队列而更新自己的数据和报表（这个过程可以是非实时、批量的，后台业务可以随时下线以更新）
- 当运营人员使用后台时，向后台业务进程请求数据

以上是解决了主流、高频的需求。还有一些低频的需求需要修改游戏中的数据，比如更新公告，赠送玩家金币，给玩家发邮件等。对于此类需求，游戏后端以HTTP形式提供实时API给后台业务进程调用。

#### 事件格式
游戏push给后台的事件都是json串。必然包含的字段有：
- typ: 事件类型
- etime: 在游戏后端事件发生的时间
- usn: 此事件是与哪个玩家挂钩

大部分事件还会有 detail字段，包含详细信息。

两个简单的例子：
```json
{  "typ":"new_user",  "ctime":1515904931, "usn":100000, "detail":{"ver":1,"isguest":1,"ban_score":0,"ctime":1517038051,"RMB":0,"ban_gold":10000,"usn":100000,"gold":0,"nick":"default user","ban_diamond":10,"diamond":0,"score":0,"credit":0,"vip_level":1} }
{  "typ":"login",  "ctime":1515905031, "usn":100000 }
```

#### 游戏的六种货币
|字段名|表示货币|
| :------: | :---------: |
|diamond|钻石|
|gold|金币|
|score|分|
|ban_diamond|绑钻|
|ban_gold|绑金|
|ban_score|绑分|

在用户数据里面的、邮件里的、或其他json里的，只要是表示这六种货币的，统一用这相同的字段名表示。

### 平台给后台开放的API
- 接口是HTTP或HTTPS协议。原则上都是POST请求
- 参数都封装成一个json串，在请求的Body里（而不是在uri里）
- 返回标准HTTP Response

##### /api/gmtool/send_mail_single
给指定usn的单个用户发游戏内邮件。

（注：游戏平台这边不支持带条件的过滤出玩家然后群发邮件，因为平台这边的数据库不是SQL。如果要群发，需要后台这边重复调用此接口）

参数： app_secure：固定值 cf7jvlKrhvCIqrfJM6cp（下同）  usn：指定用户usn  mail：json串，邮件内容，里面可夹带货币，夹带货币的话字段名见前面的表格   例子：
```json
{"usn":1, "app_secure":"cf7jvlKrhvCIqrfJM6cp", "mail":{"title":"标题", "ban_diamond":1000} }
```

##### /api/gmtool/bestow_noui
给指定usn的单个用户直接充值某种货币（与邮件不同，用户这边没有任何UI）

参数： app_secure  usn：指定用户usn    其他：至少带六种货币中的一种（可带多种）  例子：
```json
{"usn":1, "app_secure":"cf7jvlKrhvCIqrfJM6cp", "diamond":1000, "gold":2000}
```

#### 事件类型
#####  login_openID
表示用户登录。每次用户开启客户端都至少对应一个此事件。例子：
```json
{"detail":{"key":"pNh6Z9HirHAGyUm4"},"etime":1517755238,"typ":"login_openID","usn":"100102"}
```

#####  login_wx
表示用户调起微信登录（注：按照设计用户不会每次登录都调起微信）。  例子：
```json
{"detail":{"key":"wxopenidpNh6Z9HirHAGyUm4"},"etime":1517755238,"typ":"login_wx","usn":"100102"}
```

#####  login_qq
表示用户调起QQ登录（注：按照设计用户不会每次登录都调起QQ）。  例子：
```json
{"detail":{"key":"qqopenidpNh6Z9HirHAGyUm4"},"etime":1517755238,"typ":"login_QQ","usn":"100102"}
```

#####  logout
表示用户关闭客户端（注：此事件有没有完全取决于客户端有没有向平台发送logout）。  例子：
```json
{"etime":1517755238,"typ":"logout","usn":"100102"}
```

#####  new_user
表示创建新用户（注：这之后应该会有此用户登录的事件，这两者不冲突）。  例子：
```json
{"detail":{"ver":1,"ban_score":0,"isguest":1,"RMB":0,"ban_gold":10000,"ctime":1517750418,"gold":0,"usn":100097,"ban_diamond":10,"diamond":0,"score":0,"credit":0,"nick":"default user"},"etime":1517750418,"typ":"new_user","usn":100097}
```

#####  pay_begin
表示用户生成预订单（生成预订单在正式支付之前，有这一步不一定真的会支付）。  例子：
```json
{"detail":{"attach":"cheshi2","sign":"1352dd2e3a010336168419f8a773d711","app_id":"10000","userIdentity":"USER100000","total_fee":"88","notify_url":"http:\\/\\/www.baidu.com","para_id":"10000","mch_app_id":"http:\\/\\/www.baidu.com","stat":1,"order_no":"HUS00000011","version":"Pa2.5","pay_type":"0","child_para_id":"1","usn":"100000","mch_app_name":"cheshi2","device_id":"1","body":"cheshi2","mch_create_ip":"211.161.244.185"},"etime":1517816183,"typ":"pay_begin","usn":"100000"}
```
注：对于平台来说，detail里最重要的字段有： total_fee-支付RMB数额（100是1元）  order_no-订单号唯一编号  mch_app_name-商品ID
