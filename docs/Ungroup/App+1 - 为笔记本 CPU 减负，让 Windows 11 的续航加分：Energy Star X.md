> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/75565)

> 让 Windows 11 的任务管理器飘满文明的「小绿叶」。

打开 Windows 11 的任务管理器，你会发现微软自家的 Edge 浏览器和其他任务进程在「状态」一栏中的显示内容略有差别：Edge 浏览器的进程状态中有一个绿叶标志；如果你在 Edge 浏览器中开启了标签页休眠等功能，展开 Edge 的进程详情后还会进一步看到「效率模式」四个大字。

![](https://cdn.sspai.com/editor/u_/ccbchttb34td4gso3tu0)Edge 默认开启了效率模式

除了极少数情况下的后台刷新和音视频播放，处于效率模式下的 Edge 用起来似乎也并无不便。这或许正是近两年 Chrome 也开始重视起了性能和续航表现，但很多人依然觉得 Edge 在 Windows 笔记本设备上用起来更省电的主要原因之一。

**那什么是效率模式？**

从 Windows 10 的 21359 预览体验版开始，微软引入了一项名为 EcoQoS 的新机制，和其他领域中[服务质量](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8A%A1%E8%B4%A8%E9%87%8F)管控机制的存在意义类似，EcoQoS 的出现也是为了让 Windows 更聪明地进行资源调度，更明确地说，是在那些对性能或延迟要求不高的工作中，让 CPU 以最节能的方式高效完成任务。

![](https://cdn.sspai.com/editor/u_/ccbchu5b34td4gso3tug)CPU 频率提升的同时功耗也会指数级提升 | 图：微软

EcoQoS 一方面更节能，更符合微软[可持续应用开发](https://devblogs.microsoft.com/sustainable-software/)和 [2023 负排放计划](https://blogs.microsoft.com/blog/2020/01/16/microsoft-will-be-carbon-negative-by-2030/)，另一方面也能让我们的笔记本设备在实际运行过程中的发热更少、风扇运行噪声也更小；对开发者而言，CPU 运行功耗的降低也将减少发热降频带来的影响，理论上来说反而还可以提高应用在并发工作负载中的性能表现。

适配方法也很简单，开发者只需要通过 [SetProcessInformation](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setprocessinformation) 和 [SetThreadInformation](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadinformation) 两组 API 接口的适配，来向 Windows 系统声明自己的应用有 EcoQoS 需求即可。Windows 会向那些声明了 EcoQoS 需求的应用安排最合适的 CPU 核心并调配最合适的时钟频率，运行在 EcoQoS 状态下的应用，也会在任务管理器中获得一个绿叶图标并打上「效率模式」的标识——就像文章开头提到的 Edge 浏览器那样。

**不过 EcoQoS 也是有门槛的**。

一方面，EcoQoS 需要微软和芯片厂商对 CPU 功耗和电源管理进行联合调校，除了高通桌面处理器，英特尔第 10 代酷睿、AMD Ryzen 5000 及更新型号的处理器之外，更老的处理器型号并不支持 EcoQoS 的相关 API 和新特性；

另一方面，这种需要开发者主动声明的适配方法也注定是「全凭自觉」的，比起呼吁开发者根据实际情况为自己的应用适配 EcoQoS，微软或许更应该教育开发者大部分任务其实都没必要让 CPU 满载去跑。

这也就导致我们打开 Windows 的任务管理器之后，能看到的、开启了 EcoQoS 效率模式的应用寥寥无几。

虽然在部分设备上我们可以在部分进程的右键菜单中手动开启效率模式，但这种方式操作起来还是相当繁琐的。所以如果你的处理器比较新，符合 EcoQoS 的硬件要求，并且想为 Windows 设备（尤其是 Windows 笔记本）上的应用开启更广泛的效率模式支持，以 [EnergyStar 项目](https://github.com/imbushuo/EnergyStar)为基础开发的 GUI 应用 Energy Star X 是个更不错的选择。

![](https://cdn.sspai.com/editor/u_/ccbchudb34td4o4tsapg)手动开启相对麻烦

Energy Star X 完全免费，你可以从[微软应用商店](https://apps.microsoft.com/store/detail/energy-star-x/9NF7JTB3B17P)或 [GitHub](https://github.com/ArakawaHenri/EnergyStarX/releases) 进行下载。它利用上面提到的 EcoQoS API 来限制非活动进程和后台应用的资源占用，硬件要求和微软官方 EcoQoS 要求一致，系统版本这边开发者则更建议在 Windows 11 22H2 (Build 22621) 及以上版本中使用，在 Windows 11 21H2 (Build 22000) 及以上版本表现可能会受到一定影响。

![](https://cdn.sspai.com/editor/u_/ccbchudb34td4gso3tv0)Energy Star X 主界面

Energy Star X 最重要的作用是将 EcoQoS 效率模式的开启权限由开发者声明变成了大范围强制生效，默认情况下会限制非活动进程和后台应用的 CPU 资源占用，让任务管理器在 Energy Star X 启动后飘满文明的「小绿叶」。

![](https://cdn.sspai.com/editor/u_/ccbchulb34td42iniq8g)开启 Energy Star X 后的任务管理器

同时，Energy Star X 又允许我们根据实际情况自定义：在主界面右上角的齿轮设置中找到「编辑排除进程列表」，然后将需要排除的进程名称直接填入并保存即可生效。

![](https://cdn.sspai.com/editor/u_/ccbchutb34td4o4tsaq0)排除进程列表编辑

**至于哪些应用适合启用、哪些应用又应该排除这个问题**，根据微软的官方文档，EcoQoS 非常适合那些需要优先考虑能效的后台服务，比如更新、同步以及索引服务等等，Adobe CC、iTunes 客户端等应用的更新服务都可以放心加入。至于对即时 CPU 资源要求较高、较密集的应用，比如游戏进程，如果你不放心或者常常放在后台~摸鱼~挂机，也可以在应用管理器中找到对应的进程名称并将它们填写在排除列表内。

另外，虽然理论上来说笔记本设备在使用 Energy Star X 后的续航表现和发热控制提升更明显，但台式机同样也可以通过这款小工具开源节流——只需要在 Energy Star X 的设置中打开「插电时限制后台进程」开关即可。

欢迎大家在评论区分享你的使用反馈。

**参考链接：**

1.  [Introducing EcoQoS](https://devblogs.microsoft.com/performance-diagnostics/introducing-ecoqos/)
2.  [Energy Star](https://github.com/imbushuo/EnergyStar)
3.  [Energy Star X - GitHub](https://github.com/ArakawaHenri/EnergyStarX)
4.  [Energy Star X - Microsoft Store](https://apps.microsoft.com/store/detail/energy-star-x/9NF7JTB3B17P)

> 暑期征文 [数字文具盒](https://sspai.com/post/74751) 火热征稿中，分享学习方法，拿走现金奖励 🎓

> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀