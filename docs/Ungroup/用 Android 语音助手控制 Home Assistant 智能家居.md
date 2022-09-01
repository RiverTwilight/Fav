> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/70970)

> 一部旧手机替代蓝牙音箱，只要能接入 Home Assistant 的设备都可以语音控制。

**Matrix 首页推荐**

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。

文章代表作者个人观点，少数派仅对标题和排版略作修改。

如今智能家居给我们的生活带来了很多便利，可以说是不用不知道，一用回不去。不少家电的操控也都是用嘴就能完成，甚至还能设置条件自动化执行。

对我自己而言，目前米家 app、Home Assistant 我都在同时使用，Home Assistant 主要用来弥补米家的一些不足，也避免自己被米家生态困住。但在 Home Assistant 下要实现语音控制却非常麻烦，除了要安装诸如天猫精灵之类的接入设备，后台设置也不简单。一番摸索之后想到用手机自带语音助手操控 Home Assistant 执行动作的思路，如此一来只需要一台旧手机（**无论什么品牌，甚至没有自带语音助手都行**）作为语音接入就能对智能家居进行控制，连天猫精灵、小米蓝牙音箱都省了。

不过需要通过语音助手实现的每一条动作，都必须单独创建一条自动化。如果你有类似的需求并且有时间动手，不妨跟随本文一起试试。

实现思路
----

我们需要用到的工具包括一部旧手机或旧平板、[FV 悬浮球](https://play.google.com/store/apps/details?id=com.fooview.android.fooview) 或 [通知滤盒](https://sspai.com/post/59502)，以及一台安装了 Home Assistant 的设备（要是你愿意折腾，Home Assistant 其实也可以一同装在旧手机中）。

执行思路如下：

1.  Home Assistant 中预先创建自动化，触发条件设置为 webhook
2.  手机通过语音助手识别我的语音命令
3.  根据我的语音命令创建一条通知
4.  通过 FV 悬浮球的自动任务或者通知滤盒获取该通知
5.  根据通知内容发送对应 HTTP 请求到 Home Assistant
6.  Home Assistant 收到发送的请求并执行对应自动化。

如果你是用的是 iOS 设备，那只需要用 Home Assistant 安装 [HomeBridge](https://sspai.com/post/59636) 插件，就能接入 HomeKit 用 Siri 来控制智能家居了。

另外 iOS 的捷径可以直接进行 SSH 连接到安装了 Home Assistant 的设备执行 shell 命令，通过 shell 命令不仅可以实现远程唤醒、关机之类的操作，也可以执行 curl 命令来发送 HTTP 请求给任意设备。这里就不展开讲了，有需要的可以在评论区留言。

操作步骤
----

Home Assistant 平台的安装以及如何将米家设备接入，我这里就直接略过了，少数派已经有过相关的文章。

**关联阅读：**[不写一行代码，轻松将米家智能家居接入 HomeKit](https://sspai.com/post/70089)

直接进入正题，这里直接以这个 Gousund 智能插座为例（我的备注是床头灯）来操作。Gousund 智能插座接入 Home Assistant 平台后，可以直接从网页端控制。

![](https://cdn.sspai.com/2022/01/15/article/6ac0125d6ca47bdb1ff1bfacd49533fb)

### Home Assistant 中创建自动化

Home Assistant 中可以自行创建自动化项目，配置触发条件（比如时间、温度、地点等等）及执行动作（比如开关灯具、空调等等）。当条件满足时，对应动作就会自己执行，也就是 if then 结构。

而这个项目以 webhook（收到对应的 HTTP 请求）作为触发条件来执行自定义的动作。创建自动化的方法如下：

![](https://cdn.sspai.com/2022/01/18/7f001d9e457e04ae20236183118fe756.png)

*   自动化的名称可以随便写
*   触发条件选择 webhook
*   webhook ID 填个好记的 `chuangtoudeng`(记住这个 webhook ID 后面要用)
*   动作就是切换床头灯开关

这条自动化的含义就是：当 Home Assistant 接受到正确的 HTTP 请求后，执行切换智能插座开关状态的动作。

注意：Home Assistant 的 webhook 触发只支持 post 请求，所以请求方式必须选择 post，而请求的内容是：

`http://[Home Assistant的ip地址]:[部署Home Assistant的端口号]/api/webhook/[webhook ID]`

![](https://cdn.sspai.com/2022/01/18/c741ec12de79cca34aff26f9664442ee.png)详细配置参考

关于 HTTP 请求的具体的使用方法后文会有详细介绍，到这里 Home Assistant 端设置完成，接下来是手机端配置。

### 语音助手配置

语音助手在这项目用到的功能就是根据我说出的语音创建对应的通知。这里以华为平板为例（非华为系手机操作方法见后文），打开智慧生活 app，在场景中如图添加：

![](https://cdn.sspai.com/2022/01/18/86c205cd3df4d6903af46992220d8bc4.png)

如此一来，当我呼出语音助手对「小艺」说「打开床头灯」时，系统就会自动创建一条内容为「打开床头灯」的通知。

因为手上没有那么多其他品牌手机，这里只能简单介绍一下非华为系手机的解决方案：

在三星手机上，打开 Bixby 日常，添加一条日常程序，`如果` 里就选手动启动，`则` 里面就设置创建通知。

![](https://cdn.sspai.com/2022/01/15/article/a72a14907e77b1a7c7a7b31426b76f84)

然后再到 Bixby 里面添加一条语音命令来调用，内容就写「开始 XXX 日常程序」，对应上面日常里的名称（我这里用的网图，我自己的 S9 无法更新到 One UI 3.x 没法这样操作）。

![](https://cdn.sspai.com/2022/01/15/article/40d790f61ec520bc59e5b81c11f2bc68)

在 OPPO 手机上则是到小布指令里添加一个提醒，然后点到小布训练里面添加语音命令。

![](https://cdn.sspai.com/2022/01/15/article/ab87dfe41e45db1a16cb4f3e4c71a181)

对小米手机或者是根本没有自带语音助手的手机而言，可以下载并安装小爱同学 app 和米家 app。小米还是比较良心的，无论你是不是小米手机，都可以安装小爱同学，只是非 MIUI 的手机功能上有所欠缺。然后打开米家 app，并在米家 app 的场景页中创建一条手动场景，执行动作就选择向手机发送通知。最后点开用小爱来控制页面添加自己喜欢的语音命令。

![](https://cdn.sspai.com/2022/01/18/f1f31eeeca058a6838df178899c78903.jpg)

如此一来，在米家中手动点击任务或者用小爱同学都可以自动触发通知了。

### FV 悬浮球 / 通知滤盒配置

FV 悬浮球 / 通知滤盒在这个项目中发挥的作用就是当语音助手创建通知后，自动获取通知并根据通知内容发送对应的 HTTP 请求到 Home Assistant。

先介绍 FV 悬浮球。打开 FV 悬浮球创建一条自动任务：

![](https://cdn.sspai.com/2022/01/15/article/a168dd2dcd507cf30743504666226dcd)![](https://cdn.sspai.com/2022/01/15/article/be635516408f93396ec069def2246c4a)![](https://cdn.sspai.com/2022/01/15/article/f192eea801dc9a0a658d7b25895f1f87)

注意：如果用的不是华为系手机，这里需要选择对应发出通知的 app，比如三星就是 Bixby 相关的 app。如果不确定具体是哪个可以勾选多个。

![](https://cdn.sspai.com/2022/01/15/article/6f712da98fea827e02a6382c4ccc9a06)

创建好触发条件后，下面是执行的任务内容:

![](https://cdn.sspai.com/2022/01/15/article/90af3527fe84e35145994bc515faa656)

在上图中：

*   03 是获取通知的标题，通知的标题参数是 `android.title`
*   04 是获取通知的内容，通知的内容参数是 `android.text` 
*   05 是将 04 获取的通知内容作为参数传递给子任务「任务识别」

注意有些语音助手创建的通知内容会在标题里，有的则是放在内容里，这里需要根据不同情况选择将 03 还是 04 作为参数传递到子任务。比如米家 app 就非常奇葩，通过米家 app 场景生成的通知内容之前是在通知的标题里，7.0 版本开始变成在通知的内容里了……

最后我们创建子任务「任务识别」。

![](https://cdn.sspai.com/2022/01/15/article/053977c621a9cb787f04a1346244ade9)

*   任务中创建一个分支，内容为获取外部分享到任务的内容（也就是获取主任务传递过来的参数）
*   然后分支 1 设置成打开床头灯，动作为执行 HTTP 请求，请求方式为 post

这样只要主任务传递过来的参数是「打开床头灯」（和分支的内容相同），那就执行动作 HTTP 请求并发送给 Home Assistant。再次提醒 Home Assistant 的 webhook 触发只支持 post 请求，所以请求方式必须选择 post，而请求的内容是：

`http://[Home Assistant的ip地址]:[部署Home Assistant的端口号]/api/webhook/[webhook ID]`  
这里的 webhook ID 就是之前 Home Assistant 配置中自己填写的。这样只要当我对语音助手说出「打开床头灯」语音助手就会创建一条通知，内容为「打开床头灯」，FV 悬浮球自动任务获取该通知后就会进行 HTTP 请求，触发 Home Assistant 的自动化来打开智能插座了。

至于其他任何想用语音助手执行的动作，只要对应添加好 Home Assistant 里的自动化 2、语音助手的通知动作 3 并在 FV 自动任务中第二个子任务中添加一个分支就可以了。

### 通知滤盒配置

通知滤盒的设置就更为简单了，安装通知滤盒并配置好语音助手后，先对语音助手来一句：打开床头灯，再打开通知滤盒就能看到通知的相关信息。

![](https://cdn.sspai.com/2022/01/18/6a3245a82568d4bb0ba6da6bd83686e8.png)

然后退出，手动添加规则，这里点到增强页面就可以添加 webhook 相关规则：

![](https://cdn.sspai.com/2022/01/18/a32dbbc96175919d21bfa51c3eb1c7e0.png)

这里默认就是 post 方法发送 HTTP 请求，然后 URL 这里一样填写：

`http://[Home Assistant的ip地址]:[部署Home Assistant的端口号]/api/webhook/[webhook ID]`

小结
--

> 米家不香么？为什么非要用 Home Assistant？

看到这里其实很多人已经发现，如果家里买的都是支持米家的产品，大部分动作只要通过小米蓝牙音箱 + 米家 app 就都能实现，那为啥我要弄这么复杂呢？

通过这个方案，我连小米蓝牙音箱都能省了，一部旧手机就能替代；同时我也不想被小米捆绑，通过这个方案，只要能接入 Home Assistant 的设备我都可以控制，无需考虑是否兼容米家；另外 Home Assistant 的功能更多，让我可以制定更多自定义自动任务，比如通过 webhook 触发动作执行 shell 命令来远程唤醒 / 关机等等。

通过 Home Assistant 还可以弥补米家 app 自身一些不足，这也是最主要的原因。我自己就遇到米家场景中间隔 1 秒的动作执行有问题的情况：家里爸妈一直要看 IPTV，而电信给的 IPTV 机顶盒只能用红外遥控器控制，我想用语音控制小米的万能红外遥控器来调频道，于是红外对码完成后，在米家的场景中设置执行动作「按下频道 1 > 间隔 1 秒 > 按下频道 0 > 间隔 1 秒 > 按下频道 7」(对应频道 107)，实际执行的时候经常变成 170 或者先按 10、等频道都跳转了才按 7，不设间隔时间顺序就会乱，间隔 2 秒就又太长，怎么试都没用。

最后我把小米红外遥控器接入了 Home Assistant，Home Assistant 自动化中的间隔 1 秒执行则非常准确，一次都没翻过车。

> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，了解更妙的数字生活 🍃

> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀