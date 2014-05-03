title: "[SIP00]: SIP Learning"
date: 2014-04-29 19:50:05
tags: SIP
---

SIP
---------------------------
  Session Initiation Protocol
---------------------------

  create, manage and terminate sessions in an IP based network.

##SIP 可以干什么？		

    sip 只用于建立,修改和结束的sessions，**4个主要目的**：
	
1. 建立用户的location信息：把用户名和地址绑定起来：register
2. 提供一个共同准则的Session，让所有同意这些准则的用户聚在一起
3. 管理call management,加入，删除，改变参与者的状态
4. 可以改变进行中的session:cancel

##SIP 由哪些组成？
###1 UA 通常会把UAC,UAS集成在一起
- User Agent Client (UAC) : It generates requests and send those to servers.
- User Agent Server (UAS) : It gets requests, processes those requests and generate responses.
###2 Client
使用设备的终端：IP phone,PC:是发给UAC的前做的工作
###3 Servers
- Proxy Server: 最常见的一种server:最初的requests里面的消息不能准备的定位到接收者的位置, 所以发给了Proxy Servers去找到准备位置
- Redirect Server：重定向server:当前的router 找不到目的地时：返回给client要重新router找：比如：你的上海注册的电话，跑到广州去了，然后朋友打电话给你就要重定向操作
- Registrar：所有的用户都会注册到Registrar上，[遵守统一的Url方法](http://wiki.mbalib.com/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6)在Internet上所有资源都有一个独一无二的URL地址。（虽然不注册也可以发signalling，但通常都是要注册的）
- Location Server: 真正存放Registrar的地址的地方


##SIP有哪些常用的方法(Method)
* INVITE:发出邀请:与ACK配合建立session

 1. Invite 里面包含了SDP(会话描述）
 2. 如果已在session中，也可以用re-invite来调整会话（先ALice和Bob在语音会话，然后聊着聊着就又加入了视频会话）
* ACK:Acknowledgement 确认
 	与invite形成一个三次握手：Invite ->最终应答(200ok)->ACK
    
	**[Question]为什么要三次握手?**
 
	*Note* :Invite是Method里面唯一一个三次握手（其它都是用请求->应答模式)

  1. Invite建立session的时间很长，要用User来最终确定建立会话
  2. 使派生代理成为可能，Client会从不同的服务器得到应答，向每个发送应答的服务器发Ack请求来保证在UDP等不可靠协议的SIP操作。（如果其中一个成功了，让其它的会话也能清理干净）
  3. Alice 发Invite给Bob---> Bob没有收到--->Alice 再发Invite--->Bob收到返回200ok--->这时200ok在网络中又丢失了（悲剧）--->Alice没有收到200ok，会认为没有Invite成功，又发Invite---->直到成功收到200ok--->会话还没有建立（再发个ACK确认）这个例子的点在于：如果Alice 发了Invite没有收到200ok后就直接关掉了，Bob在这里发出200ok后，发现Alice没有再发Invite了，Bob就认为会话建立成功了，天哪，只是Alice不线了，如果再加上一个ACK就可以避免这种情况了

* Cancel 取消一个没有最终应答的请求。
  1. 发生一个transcation里面，请求取消已最终应答的Invite是无效的，比如：Alice 对Bob 发出Invite，Bob一直振铃（1XX,临时应答）没有接，然后Alice就发Cancel取消Invite。

		*Attaction*服务对Cancel处理完后还是会发200ok的让Invite的三次握手还是正常进行
   2.Cancel在派生代理里面的应用：Alice 给Bob发Invit-->Registar发现有3个地址可用：Bob@home,Bob@office,Bob@outside-->派生代理会向三个派发-->其中Bod@home返回200ok-->200ok沿路返回派生代理时，派生代理会给余下2个请求发cancel让他们停止（Invite事务）振铃

* Bye 请求放弃session
  1. 会话只有2方时，其中一方放弃就会terminate session
  2. 会话多方时，只是自己退出会话，其它不受影响。（实现中，其它人离开通常不做Bye处理）
  
* Register 注册
  1. 用户发出Register告诉服务器用户当前的位置（可多个）所以常和位置定位服务绑定在一起
  2. 上面说的是最简单的注册方式，基本上没有人用，因为网络总是不安全的，还要加入各种安全认[著名而有意思的RSA](http://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86) 
  3. UAC 发给UAS Register--->UAS 返回401Unauthorized(表明UAC要认证），返回时会加 proxy-Authenticate，noce(这个就是用RSA生成的钥匙）--->UAC得到这个消息后再加上Username,Url等发给专门注册的UAS，然后这个UAS会去根据这个钥匙再复杂地认证是不是真的-->然后就注册成功{ok,200,Opt}.bababa.....实际应用很复杂的，不要想帮简单了。


* OPTIONS: 请求服务器的性能，查看对方支持什么？				
    
    例如：请求后知道服务器支持SDP,支持哪一种压缩方式，那么客户端就可以发送相应的压缩SDP,从而节省带宽。
##SIP Session
 以下只是一个典型的SIP case(**实际复杂多了**）
![scession](http://www.siptutorial.net/SIP/images/example.gif)

1xx = 通知性应答   


    100 正在尝试
    180 正在拨打
    181 正被转接
    182 正在排队
    183 通话进展 


2xx = 成功应答


    200 OK
    202 被接受：用于转介 


3xx = 转接应答


    300 多项选择
    301 被永久迁移
    302 被暂时迁移
    305 使用代理服务器
    380 替代服务 


4xx = 呼叫失败


    400 呼叫不当
    401 未经授权：只供注册机构使用，代理服务器应使用代理服务器授权407
    402 要求付费（预订为将来使用)
    403 被禁止的
    404 未发现：未发现用户
    405 不允许的方法
    406 不可接受
    407 需要代理服务器授权
    408 呼叫超时：在预定时间内无法找到用户
    410 已消失：用户曾经存在，但已从此处消失
    413 呼叫实体过大
    414 呼叫URI过长
    415 不支持的媒体类型
    416 不支持的URI方案
    420 不当扩展：使用了不当SIP协议扩展，服务器无法理解该扩展
    421 需要扩展
    423 时间间隔过短
    480 暂时不可使用
    481 通话/事务不存在
    482 检测到循环
    483 跳数过多
    484 地址不全
    485 模糊不清
    486 此处太忙
    487 呼叫被终止
    488 此处不可接受
    491 呼叫待批
    493 无法解读：无法解读 S/MIME文体部分 


5xx = 服务器失败


    500 服务器内部错误
    501 无法实施：SIP呼叫方法在此处无法实施
    502 不当网关
    503 服务不可使用
    504 服务器超时
    505 不支持该版本：服务器不支持SIP协议的这个版本
    513 消息过长 


6xx = 全局失败


    600 各处均忙
    603 拒绝
    604 无处存在
    606 不可使用 

## SIP 请求消息格式

    INVITE sip:user2@server2.com SIP/2.0
    Via: SIP/2.0/UDP pc33.server1.com;branch=z9hG4bK776asdhds Max-Forwards: 70
    To: user2 <sip:user2@server2.com>
    From: user1 <sip:user1@server1.com>;tag=1928301774
    Call-ID: a84b4c76e66710@pc33.server1.com
    CSeq: 314159 INVITE
    Contact: <sip:user1@pc33.server1.com>
    Content-Type: application/sdp
    Content-Length: 142
    
    ---- User1 Message Body Not Shown ---- 
上面是一个INVITE 的Request最基本消息（会话描述的协议没有写出来哦,少年是不是感觉和http一个样子）：
 一个SIP请求由**一个请求行(Request_line），几个标题头(Headers）,一个空行(Empty line),一个消息体(Message body)**,消息体是可选的，一些请求可以不带。
 
*Request_line*:

	方法（INVITE) 请求URL(sip:user2@server2.com) 版本号(SIP/2.0)

##SIP 应答消息格式

    SIP/2.0 200 OK
    Via: SIP/2.0/UDP site4.server2.com;branch=z9hG4bKnashds8;received=192.0.2.3
    Via: SIP/2.0/UDP site3.server1.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
    Via: SIP/2.0/UDP pc33.server1.com;branch=z9hG4bK776asdhds;received=192.0.2.1
    To: user2 <sip:user2@server2.com>;tag=a6c85cf
    From: user1 <sip:user1@server1.com>;tag=1928301774
    Call-ID: a84b4c76e66710@pc33.server1.com
    CSeq: 314159 INVITE
    Contact: <sip:user2@192.0.2.4>
    Content-Type: application/sdp
    Content-Length: 131
    
    ---- User2 Message Body Not Shown ---- 
一个应答消息由**一个状态行(Status line),几个标题头(Headers),一个空行(Empty line),一个消息体(Message body)**，消息体是可选的，一些应答可以不带。

*Status line*

		协议版本(SIP/2.0) 状态码(200) 一些原语(OK)
原语只给人阅读用，于机器没一点用。

*Note* 
 
我们只要保证可靠的最终应答(如：200），临时的可以不在乎，因为我们只关心session是否被建立，失败是由于什么原因,而不关心会话是怎么建立的

##SIP各标题头的作用

1. From:user1 <sip:user1@server1.com>;tag=1928301774
	
	人名+他的SIP Url+tag=

2. CallID:a84b4c76e66710@pc33.server1.com

	不同的事务有不同的callid,Bob和Alice在进行棋盘会话Call_id1,这里2人想语音会话，发Invite里Callid2，这callid1=/=callid2用于标识不同的transcation!

3. Content: <sip:user2@192.0.2.4>

	根据这个可以直接找到用户的URL,这个特性非常重要，因为他会话建立后，再建立下一个同样的会话就可以不经过第一次那么麻烦的寻求了，这理论是很美好的，但现实中，为了计费，绝对不可能让content给client给存起来(这样以后client就可以直接联系，不经对服务器），所以这个东西一定存在server上，client如果需求，就会发请求，由server来处理，但也不用像第一次复杂（因为Server是知道Bob在哪的）
4. CSeq: 314159 INVITE

	- 一个整数字段 方法名，整数部用于同一session(CallID决定)中不同的请求排序，它会与将请求和应答想匹配：比如：Alice 发1 Invite 没返回--->再发 2 Invite--->没返回--->再发3 Invite--->这时返回了2 Invite就知道是第2个请求得到了响应（这个数是一直递增1的）。
	- Ack的CSeq:这个是与Invite里面的一样的，这使代理为非成功最终应答产生Ack时不用再建立新的CSeq,保证唯一性，只用client代理创建哦。
	- Cancel的CSeq:这个也是与Invite里面的一样的，这也是为什么CSeq里面要加Method的原因，如果不加，client就不知道这个是cancel还是invite的应答了。

5. 