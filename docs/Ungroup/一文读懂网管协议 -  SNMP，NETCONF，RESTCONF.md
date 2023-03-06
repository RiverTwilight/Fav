> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/michael9/p/14432935.html)

本文篇幅较长，主要涉及以下内容：

*   介绍传统 CLI 配置网络设备存在的挑战，网管协议出现的背景
    
*   SNMP 原理，交互过程，以及 trade-off
    
*   NETCONF 架构，交互过程
    
*   RESTCONF 架构，和 NETCONF 的对比
    

随着 5G 的大火，SDN, NFV 等概念被频繁提及。想要更好的理解这些概念，网络协议自然是对必不缺少的一环。

拿 SDN 来说，全称为 Software Defined Networking - 软件定义网络。从传统网络来说，整体采用分布式的架构，控制平面和转发平面都位于同一台设备上。在运维，以及灵活性上都有着不小的挑战。

而 SDN 的出现，将控制平面和转发平面解耦，并将所有的设备统一管理起来，使得网络具有了可编程的能力，从面相服务的角度，根据业务需要，实时动态的调整设备的配置或状态，大大降低了管理难度。

那么 NETCONF，RESTCONF 这样的协议又是起到怎样的作用呢？

这就要从 SDN 架构说起，SDN 主要有三个角色，SDN 应用，SDN 控制，网络设备。

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210735518-1221306337.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210735518-1221306337.jpg)

SDN 应用一般会通过 HTTP 的方式，调用 SDN 控制器暴露的接口，而 SDN 控制器，会通过 NETCONF，RESTCONF 类似的协议，与设备进行交互，进行业务的下发。

可见，NETCONF 类似的协议起到和设备直接交互的作用。

还有最近 DEVOPS 概念的流行，也强调从传统人工 CLI 配置，过度到自动化的网络配置。而 NETCONF 类似协议，就让这些自动化操作成为可能，比如现在的 ANSIBLE，Python 的各种类库。

下面是网管协议的实际用例，可以看到涵盖的范围已经非常之广：

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210753260-1558750223.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210753260-1558750223.jpg)

下面就从传统 CLI 面临的挑战开始，来详细了解下这些发挥着重要作用的协议。

传统命令操作带来的主要挑战[#](#传统命令操作带来的主要挑战)
--------------------------------

采用传统 CLI 配置方式主要存在着如下的挑战：

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210813179-2039056232.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210813179-2039056232.jpg)

在兼容性方面，若是网络工程师的话肯定深有体会。

拿配置静态路由举例， Cisco 设备命令如下：

```
Router(config)#ip route 0.0.0.0 0.0.0.0 10.10.10.1 


```

而对于华为和华三设备来说：

```
Router(config)#p route-static 0.0.0.0 0.0.0.0 10.10.10.1 


```

有时不光是不同厂商之间的命令不同，甚至同一厂商不同型号之间的命令也不相同。

比如思科针对不同的场景就区分了不同的网络软件系统：

*   针对于企业的 IOS
*   针对于运营商的 IOS-XR
*   针对于数据中心的 NX-OS
*   以及面向下一代的 IOS-XE，将数据层面和控制解耦，底层支持 Linux.

而出错率这方面，更不必说，人工配置远没有机器配置的准确以及迅速。

而且目前的网络在规模和需求上也和之前大不一样，比如在实时性上，作为运营商需要根据业务需求动态调整策略如 EVPN，L3VPN，L2TP。传统 CLI 手工配置根本无法满足，而无法做到维护管理。而现在的常用解决方案都是利用一些现成的 SDN Controller 进行实时调整如 Cisco 的 NSO.

在数据采集方面，传统人工定时登上设备材料系统日志，分析情况，这更加无法适用。在故障响应上，数据采集，分析都存在先天的不足。比如收集效率低，数据的利用率延迟或不高。

最后从商业成本的考虑，人工的维护方案也是较高的一笔输出。

而且对于工程师而言，需要不断学习不同厂商的配置命令，学习成本很高，但意义不大。

通常来说，网络工程师在开始一个项目前，会进行如下四个部分：

1.  了解用户的需求
2.  针对用户的需求，确定相应的具体方案。
3.  根据用户的方案，查找并学习对应设备的配置命令
4.  最后申请割接窗口，准备回滚方案并实施。

在这里的第三步，其实就是一个比较耗时，但没多少意义的过程。因为在集中学习不同厂商的设备命令，但从业务考虑明明解决的都是同一个问题。

在发现 CLI 管理设备的方式出现瓶颈后，并不是马上过度到现在流行的网络自动化配置方式，而是先推出了一个叫 `Simple Network Management Protocol` - SNMP 的应用层协议，甚至在当前的一些现网中，依然被使用。

SNMP[#](#snmp)
--------------

SNMP 的出现，主要想解决两个问题：

1.  设备信息的采集
2.  使用 GUI 替代 CLI 的方式进行设备配置下发

但由于其读多写少的特点，现被广泛用于设备信息的监控和采集。

SNMP 目前共有三个版本：

*   SNMP V1，第一个版本。在管理设备上，采用明文的方式，有 `read-only`, `read-write`, `trap` 三种和设备通信的方式。
*   SNMP V2，主要改进了性能，安全性以及设备交流的方式。
*   SNMP V3，主要优化了安全性，增加了一些更强的认证流程。

### SNMP 原理[#](#snmp-原理)

SNMP 整体架构上有些类似于 Client / Server，其主要的工作组件主要有三个：

*   SNMP Manager：，主要用于管理网络中的多个设备，对其进行读和写的操作。类似于 Server.
    
*   SNMP Agent：运行在网络设备上，通常都需要手动开启。作为 SNMP 代理，在收到 SNMP Manager 发出请求后，对请求的内容进行解析，然后对设备进行配置，将配置的结果作为 Response 回复给 Manager.
    
*   SNMP MIB: MIB - Management Information Base 全称为信息管理库。可以将其理解成用于交互的一种数据模型，也就是交互的规则。MIB 同样存在于网络设备中。定义和描述了如何管理设备上的资源。Manager 和 Agent 之间的交流的信息就是 MIB 的内容。
    

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)](media/16088761172467/16120068927911.jpg)

可以看到一个 Manager 可以管理网络中的多个设备。而每台设备上运行着 SNMP Agent 用于和 Manger 信息交互，交流的内容需要符合 MIB 的规范。

看到这，可能对 MIB 这个概念还是有些模糊。这样，我们先不从最后的结果来分析这个组件的作用，而是从设计的角度，来说一下推导下为什么要有 MIB 这个东西？

这里想要实现的是通过 Manager 去管理网络上的 Agent（其实就是管理设备）。那么如何管理呢，比如 Manager 想要获取 Agent1 的 GigabitEthernet0/0/0/1 的 IP 地址。

这时就需要在 Agent1 上先约定好一个内容，比如当 Agent 接收到 `1.1` 这个字符串时，就会将接口的信息返回给 Manager.

之后如果 Manager 发送 `1.1` 就能获取到接口的信息了，但发送别的内容，Agent 是无法识别并工作的。MIB 本质就是这样，确定了如 `1.1` 这样的一组规则，去规范信息交互的访问方式。

其实，这里的 `1.1` 就是 MIB 中的一个对象，在 MIB 中还以层级的方式存在着许多这样的对象，将网络的设备的资源抽象成形如 `1.1` 对象。通过这些对象，Manger 和 Agent 就可以实现很好的交流了。

真正的 MIB 类似与下图，而这里形如 `1.3.6.1.1.1.2` 这样连接起来的字符串称为 ASN，其实就是对应了设备上的各种资源，Manager 和 Agent 也通过它们进行交流。

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210834925-565780461.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210834925-565780461.jpg)

### SNMP 操作[#](#snmp-操作)

SNMP Manger 和 SNMP Agent 间的交互主要有三种类型：

*   SNMP Get
*   SNMP SET
*   SNMP Notifications

SNMP Get：主要是检索设备的信息，Get 一种有三种类型：

*   GET - 从 SNMP agent 获取固定的对象。
*   GETNEXT - 检索当前对象的后一个对象，由于 MIB 本身层级的树形结构，存在后继。
*   GETBULK - 获取一组固定的对象。

SNMP SET：主要是修改 MIB 中的对象，进而修改设备的配置。

SBMP Notifications：是 SNMP 的主要特性，之前的 GET 和 SET 是属于拉的操作，而 SNMP 正好相反，类似于推的操作，可以由 agent 发起，将一些信息 push 到 SNMP Manager 上。类似于 Web Socket. 主要用于通知如认证失败，重启，断开连接等事件。

Notifications 主要有两种形式：Traps 和 Informs. 两者间的不同主要在于可靠性，agent 在产生 Informs 给 Manager 后，如果发送失败，会重新发送。Manager 收到后，需要回复确认给 agent。

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210852067-1262313725.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210852067-1262313725.jpg)

而 Traps 不同，无论消息发送是否成功，Manager 都不需要回复。

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210904446-552172757.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210904446-552172757.jpg)

### SNMP 缺点[#](#snmp-缺点)

虽然 SNMP 的出现，在一定程度上解决了网络设备的管理问题。但面对现代大规模的网络来说，依然有着很多挑战：

1.  性能不足，在下发和读取配置时，采用依次读取，效率低。
2.  下发不足，支持写 MIB 的对象相对于读较少。
3.  不支持事务机制，在配置下发失败是，无法回滚。
4.  拓展性差，提供给外部的接口较少。
5.  模型兼容性差，MIB 库混乱，无法适配所有厂商，导致定义各种私有 MIB 库。

面对这些问题，06 年由 IETF 领导并开发出了一个新的协议 - NETCONF，网络管理协议。和 SNMP 不同，NETCONF 基于 RPC 的方式，天生就能很好的支持事务回滚等操作，从而更好地处理复杂网络的各种需求。

NETCONF[#](#netconf)
--------------------

NETCONF 协议提供了一种更简单的方式来管理（"查询，配置，修改，删除"）设备，就像数据库操作中的 DML. 同时开放了 API 接口，当想要对设备进行操作时，直接通过调用 API 进行。

对于支持 NETCONF 的设备来说，至少能开启一个或多个 session。并且在每个 session 中应用的配置更改，都可以被全局的 session 监听到。这就让一个或多个 Manager(Client) 操作同一个设备（agent）成为了可能。

相比 SNMP 而言，有着如下的优势：

1.  基于 RPC，增加了事务支持
2.  优化查询功能，增加过滤查询方式
3.  拓展性强，在其协议内部分为 4 层，各层之间相互独立
4.  更好的将配置和状态数据解耦，并区分状态数据（candidate, running, startup）
5.  易使用，结合提供的 API，实现可编程性的网络操作
6.  安全性更好，在传输层可选用 SSH，TLS 协议等。

NETCONF 采用 C/S 的架构，通过 RPC 在 client 和 server 间交流。client 可以是 Python 脚本或应用。server 一般指的是网络设备，在具体实现上有三个组件：

1.  NETCONF agent：运行在网络设备上，用于接收和处理 RPC 请求。还可主动将一些告警事件通知客户端。
    
2.  NETCONF 客户端：利用 NETCONF 协议对网络设备进行管理以及接收 agent 发出的告警通知。
    
3.  datastore：在 NETCONF 中，区分了多个不同类型的 datastore, 这些 datastore 保存着不同状态下的设备信息。
    

关于 datastore 可以将其理解成一个可以获取和存储信息的概念。在具体实现上，可以是文件，数据库，内存等等。

在 NETCONF 中常用到三类 datastore：

*   startup configuration datastore: 保存了设备启动时，加载的配置信息。
    
*   candidate configuration datastore: 保存了想要运行的配置信息，修改该数据库时，并不会影响设备的真实配置。
    
*   running configuration datastore: 保存了当前设备上正在生效的配置，修改时会影响真实的设备。
    

此外提到 datastore 就必须要提到一种数据模型语言 —— YANG，datestore 中就是以 YANG 的形式来约束配置的数据。

YANG 的出现结合上 NETCONF 和 RESTCONF 这样的协议，为自动化，可编程化的网络提供了强大的支持。YANG 的本质和之前 SNMP 中 MIB ASN 一样，作用都是以一种方式来约束数据，关于 YANG 之后会写一篇文章单独介绍。

### NETCONF 协议架构[#](#netconf-协议架构)

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210930470-515066102.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210930470-515066102.jpg)

NETCONF 分为 4 层，各层之间项目独立。

1.  内容层：

这一层包含了以 XML 或 JSON 格式的配置数据，也就是想对设备进行管理的具体内容。（由 XSD 或者 YANG 约束生成）

2.  操作层：

定义了 Client 和 Server 交互时的一系列操作方法，用于获取或修改配置数据。

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210946325-1563306962.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222210946325-1563306962.jpg)

3.  消息层：

为编码数据时，提供了一种 RPC 和通知的机制：

```
* RPC invocations(<rpc> messages)  
* RPC results(<rpc-reply> messages)
* event notifications(<notification> messages)


```

4.  传输层：

NETCONF 使用 SSH 或 TLS 协议，保证数据在 Client 和 Server 传输的安全性。

### NETCONF 交互[#](#netconf-交互)

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211001592-1596655565.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211001592-1596655565.jpg)

对于 Manager 和 Agent 来说，Session 建立会经历如下的过程：

1.  Manager 请求 NETCONF 中 SSH 子系统建立连接。
2.  Agent 回复 Hello 消息，包含本身支持的特性和能力。
3.  Manager 告知 Agent 自己所支持的特性和能力。
4.  Manager 开始发送 RPC 操作请求。
5.  Agent 回复 RPC 请求操作结果。

具体看下 NETCONF 中消息的报文结构，以修改接口配置举例：

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211020642-311677650.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211020642-311677650.jpg)

Manager 请求变更接口配置：

```
 <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <edit-config>
         <target>
           <running/>
         </target>
         <config>
           <top xmlns="http://example.com/schema/1.2/config">
             <interface>
               <name>Ethernet0/0</name>
               <mtu>1500</mtu>
             </interface>
           </top>
         </config>
       </edit-config>
</rpc>


```

Agent 回复结果：

```
 <rpc-reply message-id="101 xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <ok/>
 </rpc-reply>


```

首先可以看到，NETCONF 使用 XML 作为数据传输的格式。

第一行的 `<rpc>` 标签，表明该请求是 RPC 请求，`message-id` 属性由 client/manager 确定。Agent 在回复结果时，会带上 `message-id`，用于表示该操作的结果。

`urn:ietf:` 属性表示 XML 中的命令空间。 `base：1.0` 表示当前 NETCONF 的版本。

第二行 `<edit-config>` 指定执行 RPC 操作的内容为修改配置。

之后 `<config>` 中包裹的内容就是，想要下发的配置内容，修改 mtu 为 1500.

但这里还有一个疑问，在说到 NETCONF 时，总提到一种叫 YANG 的语言，那么它们之间的关系是什么？

在组装修改设备配置 Payload 时，这里也没有用到 YANG ？

其实，YANG 早已用到了。为什么 `<interface>` 中可以包含 `<name>` 和 `<mtu>` 属性。而不是把 `mtu` 放在和 `<interface>` 同级。

**原因就在于上面的格式，都是按照 YANG 的约束而来。**

在设备中，存在着许多的 YANG Module，这些 Module 都是由 YANG 语言编写的文件。当 `agent` 接收到 RPC 请求时，会通过 YANG Module 来校验发来的数据，格式是否合法。

**简单来说，可以把 YANG 理解成一份约束文件，里面规定着传来参数的格式，是数组，对象还是其他格式。**

至于说为什么 YANG 文件能约束 XML 的文件格式，原因在于 YANG 和 XML 之间是可以相互转换的，甚至 YANG 还可以转换成 JSON. 在之后的 RESTCONF 中会提到这一点。

到目前为止，对 NETCONF 已经有了一个大体认识：

NETCONF 的出现是为了弥补像 SNMP 这些协议的不足。更好的满足现在网络的需要，在易用性，拓展性等等方面都做出了进一步优化，从而更方便，高效的管理网络。

究其本质，NETCONF 是由多个协议组装而成。数据的产生及校验通过 YANG，数据的格式是 XML. 接口的调用通过 RPC，数据的传输通过 SSH.

### NETCONF 操作[#](#netconf-操作)

Server 端：打开设备 NETCONF

```
#  打开 netconf 
netconf-yang

# 查看 netconf 进程
show platform software yang-management


```

Client 端：测试设备 NETCONF

```
ssh admin@IP -p 830 -s netconf


```

关于 NETCONF 具体实现编程化操作，可以参见 YANG 这篇。

RESTCONF[#](#restconf)
----------------------

在谈起 RESTCONF 前，想必刚接触这个概念的人都会有这样一个疑问 RESTCONF 和 REST 到底有没有关系？

再回答这个问题前，先来回忆一下什么是 REST，以及 REST 出现的背景。

REST - Representational State Transfer，全称为表现层状态转化，是建立在 HTTP 基础上，对其进行规范的一种架构风格要求。

> 注意，REST 是一种设计的风格，而不是标准。

其认为，网络中的实体都是以资源的方式存在，但资源却存在着多种表现形式，取决于使用者的需要。比如一个用户的信息，可以用 XML, JSON, 甚至是 txt 等多种方式表现出来。将不同的网络资源转换成不同的表现形式，就是其表现层的体现。

而状态转移来说，由于 HTTP 本身是无状态的协议，所以资源的状态全都保存在服务端。当对服务端的资源进行操作时，必然存在数据状态的改变。

但由于状态的改变基于表现层，所以称为表现层状态转移。

在具体实现上，URI 定义了访问资源的具体路径，而 HTTP 中 Header 的 `Content-Type` 和 `Accept` 决定了了表现层的形式。

HTTP 中的 CURD 动作（Create，Put，Get，Delete，patch..）去改变服务端的资源状态。

比如查询书店具有的图书：

```
GET http://www.store.com/products


```

通过 REST 的方式，更合理的实现 WEB 服务之间的交互。

这时再看 RESTCONF，就很好理解了。RESTCONF 是通过 REST 来实现对网络设备管理的协议。其本质和 NETCONF 很像，使用 YANG 进行数据的定义和约束，使用 HTTP 进行交互。使用 NETCONF 中 datastore 的概念，进行信息的储存。

### RESTCONF 架构[#](#restconf-架构)

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211052995-2051697716.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211052995-2051697716.jpg)

图中很好的表示了 RESTCONF 协议组成，很形象的指出 RESTCONF = NETCONF / YANG + HTTP(s).

如果拿之前的 NETCONF 协议架构作为对比，RESTCONF 就是将：

*   内容层，同样由 YANG 约束生成。
*   RPC 消息层和操作层，换成了 HTTP 的操作层。
*   将 SSH 构成的传输层，换成了 HTTP（s）的传输方式。

### RESTCONF VS NETCONF 交互[#](#restconf-vs-netconf-交互)

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211107008-1483195081.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211107008-1483195081.jpg)

图中很好的对比了 RESTCONF 和 NETCONF 的交互过程，都是采用了 C/S 架构，在具体组件上：

<table><thead><tr><th></th><th>NETCONF</th><th>RESTCONF</th></tr></thead><tbody><tr><td>客户端</td><td>NETCONF client</td><td>HTTP Client</td></tr><tr><td>配置格式由谁约定</td><td>YANG module / XSD</td><td>YANG module</td></tr><tr><td>发送内容格式</td><td>XML</td><td>XML/JSON</td></tr><tr><td>交互方式</td><td>RPC</td><td>HTTP</td></tr><tr><td>传输协议</td><td>SSH</td><td>HTTP(s)</td></tr><tr><td>服务端</td><td>NETCONF server</td><td>HTTP server</td></tr></tbody></table>

对于操作来说，将 RPC 操作换成了 HTTP 操作：

[![](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211248990-461280776.jpg)](https://img2020.cnblogs.com/blog/1861307/202102/1861307-20210222211248990-461280776.jpg)

### RESTCONF 操作[#](#restconf-操作)

Server 端：打开设备 RESTCONF

```
#  打开 RESTCONF 
restconf-yang

# 查看 RESTCONF 进程
show platform software yang-management


```

Client 端：

由于已经采用了 REST 风格，可以利用 POSTMAN 等等 HTTP 客户端进行测试。

关于 RESTCONF 具体实现编程化操作和其 URL 的使用是非常重要的一部分，但由于其依赖 YANG 这个概念，这个后面会单独提到，可以参见 YANG 这篇。

总结[#](#总结)
----------

这篇文章，耗时很久，查阅了大量资料，完成后真的如释重负一般。

当然对网管协议也有了进一步的理解。

下面做一个简单的总结：

传统 CLI 配置方式，已经无法满足当代网络可编程化的需要，而且在兼容性，易用性，正确率存在着诸多问题，进而网管协议应运而生。

SNMP 作为推出的第一代协议，在一定程度上解决了设备管理的问题。但由于其读多写少的特点，以及在兼容性，效率，以及缺乏事务性的不足，在现网中，一般用其作为设备配置采集或监控的工具。

为了更好的满足网络的需要，第二代协议 NETCONF 出现，由于 NETCONF 天生 RPC 支持事务的特点，再加上 YANG 解决了多厂商命令兼容性的问题，现被广泛使用在各种网管平台，SDN 控制器中。

随后 HTTP REST 风格的普及，IETF 又推出了 RESTCONF 协议，将 NETCONF 和 HTTP 整合在一起，以更为流行的方式，实现对设备的管理。

参考[#](#参考)
----------

[SNMP 介绍](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/snmp/configuration/xe-16/snmp-xe-16-book/nm-snmp-cfg-snmp-support.html#GUID-633FA74F-DA24-419F-B0AC-3CD0D3FD8384)

[NETCONF - wiki](https://en.wikipedia.org/wiki/NETCONF)

[RFC6241 - NETCONF](https://tools.ietf.org/html/rfc6241)

[NETCONF datestore](https://tools.ietf.org/id/draft-ietf-netmod-revised-datastores-08.html)

[通过 NETCONF 实现网络自动化](https://medium.com/@k.okasha/network-automation-and-the-rise-of-netconf-e96cc33fe28)

[REST 介绍](http://www.ruanyifeng.com/blog/2011/09/restful.html)

[RESTCONF - RFC8040](https://tools.ietf.org/html/rfc8040)

[基于 Model 驱动的自动化网络](https://www.ciscolive.com/c/dam/r/ciscolive/emea/docs/2018/pdf/LTRCRT-2700.pdf)

[RESTCONF 介绍](https://www.ciscolive.com/c/dam/r/ciscolive/apjc/docs/2016/pdf/DEVNET-1081.pdf)

[CISCO 可编程化网络](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst4500/XE3-9-0E/15-25E/configuration/guide/xe-390-configuration/prgrmblty.html)

[SDN 和 NFV 的关系](https://time.geekbang.org/column/article/210059)