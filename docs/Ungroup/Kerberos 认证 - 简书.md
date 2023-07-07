> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/23a4e8978a30)

一、Kerberos 认证
-------------

### Kerberos 的重要性：

对我们搞 Web 的而言，弄清 Kerberos 认证过程，最有利于帮助我们理解域内的金票和银票！

### Kerberos 介绍：

在古希腊神话中 Kerberos 指的是：有着一只三头犬守护在地狱之门外, 禁止任何人类闯入地狱之中。  
而现实中的 Kerberos 是一种网络身份验证协议, 旨在通过密钥加密技术为客户端 / 服务器应用程序提供身份验证, 主要用在域环境下的身份验证。

![](http://upload-images.jianshu.io/upload_images/18786052-dd6b64ef35191574.png)

通过上图可以看到整个认证流程有三个重要的角色, 分别为 Client、Server 和 KDC。

下面介绍下几个相关的名词：

1.  访问服务的 Client；
    
2.  提供服务的 Server；
    

3.KDC（Key Distribution Center）密钥分发中心。  
在 KDC 中又分为两个部分：Authentication Service(AS, 身份验证服务) 和 Ticket Granting Service(TGS, 票据授权服务)

4.DC 是 Domain Controller 的缩写, 即域控制器; AD 是 Active Directory 的缩写, 即活动目录。  
DC 中有一个特殊用户叫做: krbtgt, 它是一个无法登录的账户, 是在创建域时系统自动创建的, 在整个 kerberos 认证中会多次用到它的 Hash 值去做验证。  
AD 会维护一个 Account Database(账户数据库). 它存储了域中所有用户的密码 Hash 和白名单。只有账户密码都在白名单中的 Client 才能申请到 TGT。

### Kerberos 粗略的验证流程：

举个简单的栗子：如果把 Kerberos 中的票据一类比作一张门禁卡, 那么 Client 端就是住客, Server 端就是房间, 而 KDC 就是小区的门禁。住客想要进入小区, 就需要手里的门禁卡与门禁想对应, 只有通过门禁的检验, 才能打开门禁进入小区。

需要注意的是, 小区门禁卡只有一张, 而 Kerberos 认证则需要两张票。

### Kerberos 详解认证流程：

当 Client 想要访问 Server 上的某个服务时, 需要先向 AS 证明自己的身份, 验证通过后 AS 会发放的一个 TGT, 随后 Client 再次向 TGS 证明自己的身份, 验证通过后 TGS 会发放一个 ST, 最后 Client 向 Server 发起认证请求, 这个过程分为三块：

Client 与 AS 的交互,  
Client 与 TGS 的交互,  
Client 与 Server 的交互。

#### 第一步，Client 与 AS 的交互：

准备：用户在 Client 中输入账号密码后，Client 会对密码进行 hash code，我们叫做 Master key。

请求：  
Client 先向 KDC 的 AS 发送 Authenticator(认证者)，我们叫它 Authenticator1，为了确保 Authenticator1 仅限于自己和 KDC 知道，Client 使用自己的 Master Key 对其的主体部分进行加密。  
其内容为：  
1. 经过 Client 用户密码 hash code(Master key) 加密的 TimeStamp(一个当前时间的时间戳)。  
2.Client 的一些信息 (info)，比如用户名。

响应：  
(1).AS 接收到 Authenticator1 后，会根据 Client 提交的用户名在 AD 中寻找是否在白名单中，然后查询到该用户名的密码，并提取到 Client 对应的 Master key，对 TimeStamp(时间戳) 进行解密，如果是一个合法的 Timestamp，就证明了 Client 提供的用户名和密码是存在 AD 中的，并且 AS 提取到的 Timestamp 不能超过 5 分钟，否则 AS 就会直接拒绝 Client 的请求。  
(2).TimeStamp 验证通过后，AS 会给 Client 发送一个由 Client 的 Master key 加密过的 Logon Session Key 和一个 TGT(client-server-ticket)。

TGT 的内容：  
经过 KDC 中的 krbtgt 的密码 HASH 加密的 Logon Session Key(登录会话密钥) 和 TimeStamp(时间戳)、TGS 会话密钥、用户信息、TGT 到期时间。

**注意**

1.  Logon Session Key 是什么：Client 向 KDC 发起对 TGT 的申请,” 我需要一张 TGT 用以申请获取用以访问所有 Server 的 Ticket”。KDC 在收到该申请请求后，生成一个用于该 Client 和 KDC 进行安全通信的 Session Key（SKDC-Client，也被称为 Logon Session Key)。这里 KDC 不会保存 SKDC-Client。  
    需要注意的是 SKDC-Client 是一个 Session Key，他具有自己的生命周期，同时 TGT 和 Session 相互关联，当 Logon Session Key 过期，TGT 也就宣告失效，此后 Client 不得不重新向 KDC 申请新的 TGT，KDC 将会生成一个不同 Session Key 和与之关联的 TGT
    
2.  第二步会有一个 Session Key ，是用于 Client 和 Server 之间通信的 Session Key（SServer-Client）
    

#### 第二步, Client 与 TGS 的交互, Client 使用 TGT 从 KDC 获得基于某个 Server 的 Ticket：

一、请求：  
Client 通过自己的 Master key 对第一部分解密获得 Logon Session Key 之后，携带着 TGT 对 TGT 发送请求。Client 是解不开 TGT 的，它作为一个 Client 通过身份验证的票提交给 TGS。

请求的内容：  
(1).TGT：Client 通过与 AS 交互获得的 TGT，TGT 被 KDC 的 Master Key 进行加密。  
(2).Authenticator2：Client 端使用 Logon Session Key 对其进行加密，Authenticator2 实际上就是关于 Client 的一些信息和当前时间的一个 Timestamp，用以证明当初 TGT 的拥有者是否就是自己。

TGS 收到 Client 请求，验证其真实身份：  
TGS 在发给 Client 真正的 Ticket 之前，先得整个 Client 提供的那个 TGT 是否是 AS 颁发给它的，于是它得通过 Client 提供的 Authenticator2 来证明。但是 Authentication2 是通过 Client 的 Logon Session Key 进行加密的，而 TGS 并没有保存这个 Logon Session Key 。所以 TGS 先得通过自己的 Master Key{krbtgt 的密码 hash 处理} 对 Client 提供的 TGT 进行解密，从而获得 Client Info 和 Logon Session Key（SKDC-Client），再通过这个 Logon Session Key 解密 Authenticator2  
获得 Client Info，对两个 Client Info 进行比较进而验证对方的真实身份

二、响应 --TGS 验证通过后发 ST(Service Ticket) 票：

响应内容：

认证通过后 TGS 生成使用 Logon Session Key（SKDC-Client）加密过用于 Client 和 Server 之间通信的 Session Key（SServer-Client），Server 的 Master Key 进行加密的 ST(Service Ticket)

(1). 经过 Logon session key 加密的 Client 和 Server 之间的 Session Key  
(2). 经过 Server 的 Master Key 进行加密的 ST(Service Ticket)。

Ticket 大体包含以下一些内容：  
Session Key（SServer-Client）  
Domain name\Client。  
Ticket 的到期时间。

Client 收到 TGS 的响应，使用 Logon session key，解密第一部分后获得 Session Key （注意区分 Logon Session Key 与 Session Key 分别是什么步骤获得的，及其的区别）。有了 Session Key 和 ST(Service Ticket)， Client 就可以直接和 Server 进行交互，而无须在通过 KDC 作中间人了。

#### 第三步，Client 与 Server 的交互 -- 双向验证：

**Server 验证 Client：**  
Client 通过与 TGS 交互获得访问 Server 的 Session Key, 然后为了证明自己就是 ST(Service Ticket) 的真正所有者, 会将 Authenticator 和时间戳提取出来, 并使用 Session Key 进行加密。最后将这个被加密过的 Authenticator3 和 ST 作为请求数据包发送给 Server。此外还包含一个 Flag 用于表示 Client 是否需要进行双向验证。

Server 接收到 Request 之后, 首先通过自己的 Master Key(krbtgt 的密码 hash 处理) 解密 ST, 从而获得 Session Key。然后通过解密出来的 Session Key 再去解密 Authenticator3 , 进而验证对方的身份。如果验证成功, 且时间戳不长于 5min, 就让 Client 访问对应的资源, 否则就会直接拒绝对方的请求。

**双向认证：**  
到目前为止，服务端已经完成了对客户端的验证，但是，整个认证过程还没有结束。接下来就是 Client 对 Server 进行验证以确保 Client 所访问的不是一个钓鱼服务.

**Client 验证 Server：**  
Server 需要将 Authenticator3 中解密出来的 Timestamp 再次用 Session Key 进行加密, 并发送给 Client。Client 再用缓存 Session Key 进行解密, 如果 Timestamp 和之前的内容完全一样, 则可以证明此时的 Server 是它想访问的 Server。

![](http://upload-images.jianshu.io/upload_images/18786052-4acd52a97eae2cff.png)

[kerberos 认证原理参考于](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fsky_jiangcheng%2Farticle%2Fdetails%2F81070240)

至此，Kerberos 认证和域内环境介绍就结束了，谢谢观看，若有疑问请留言，若有错误请指教。

推荐另一篇关于金票和银票的文章：[黄金票据和白银票据攻击原理介绍](https://www.jianshu.com/p/4936da524040)

推荐另一篇内网渗透的文章：[内网渗透](https://www.jianshu.com/p/27c3cfd4cb30)

大佬随手给个赞呗 0.0

![](http://upload-images.jianshu.io/upload_images/18786052-626404db3bb20f62.png)