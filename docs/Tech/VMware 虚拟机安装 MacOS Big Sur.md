> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/424182886)

之前完善了 vm 安装 Windows 系统的教程，今天给大家分享一个 vm 安装 MacOS 的教程，我们今天用 macOS Big Sur 版本来做教程演示。

> * 注：使用 VMware 安装 MacOS 哪怕配置给的高也会出现体验上的不佳，大家可以尽可能调高适当的配置，避免卡顿。

1、准备工作
------

废话不多说，首先我们在 VMware 里面安装 MacOS 需要做的准备工作。

①VMware Workstation 软件

（演示版本：vmware-workstation-full-16.2.0-18760230.exe）

```
【下载链接】http://ai95.microsoft-cloud.cn/d/9289114-49833935-3c06d9?p=ai95
（访问密码：ai95）

## 支持OneDrive网盘、城通网盘、阿里网盘、百度网盘等下载

```

②macOS Big Sur 系统原版镜像文件包

（演示版本：Install-macOS-Big-Sur-11.6-20G165.iso）

```
【下载链接】http://ai95.microsoft-cloud.cn/d/9289114-49836929-d652cc?p=ai95
（访问密码：ai95）

## 支持OneDrive网盘、城通网盘、阿里网盘、百度网盘等下载

```

③Unlocker v3.0.4 VMware 虚拟机 Mac OS 系统解锁工具

（演示版本：MacOS Unlocker for VMware v3.0.4（苹果解锁插件）.zip）

```
【下载链接】http://ai95.microsoft-cloud.cn/d/9289114-49833935-3c06d9?p=ai95
（访问密码：ai95）

## 支持OneDrive网盘、城通网盘、阿里网盘、百度网盘等下载

```

![](https://pic4.zhimg.com/v2-31cc32aa2d1de5fd87915d41b12703cb_b.png)

Unlocker 解锁 VMware Workstation 虚拟机 MacOS 操作系统工具可以解锁 VMware Workstation 对苹果 Mac 系统的支持。

2、解锁 Mac OS 系统
--------------

我们先来看下未解锁 Mac OS 系统状态下的 VMware，在这种状态下是无法新建和启动 MacOS 操作系统的，如下图【2-1】所示

![](https://pic1.zhimg.com/v2-3b90725f2d66f2f6006d0f39542543c8_r.jpg)

按快捷键【ctrl+shift+esc】打开任务管理器，找到所有 VMware 程序将其强制关闭

![](https://pic3.zhimg.com/v2-3678d7f0ecc6ed7f6d6f06e642429e0e_r.jpg)

然后我们解压【Unlocker v3.0.4 VMware Mac OS 系统解锁工具】至 VM 安装目录

![](https://pic4.zhimg.com/v2-0f2eff990ac259722d5074fc0cae3f1f_r.jpg)

打开【unlocker】文件夹，找到【win-install】文件，右键【以管理员身份运行】

![](https://pic1.zhimg.com/v2-41715ea40f9f9a75b01dcdea315f8184_r.jpg)

解锁工具将会自动执行，当出现【Starting VMware services...】时，说明已经成功解锁 Mac 的安装

![](https://pic3.zhimg.com/v2-5270da567837b3193256bc2968f8253e_r.jpg)

3、配置 VMware
-----------

解锁成功后，我们就可以新建虚拟机，为安装 MacOS 配置参数

![](https://pic1.zhimg.com/v2-1a7682f9cd9e7592953a156627e11b34_r.jpg)

选择最高的【硬件兼容性】

![](https://pic2.zhimg.com/v2-d53a4cc484af0a3341aeeec071366f31_r.jpg)

这里我们先选【稍后安装操作系统】，待会儿还有一些地方需要设置

![](https://pic1.zhimg.com/v2-788d3cefde912b074099e3f089da09b0_r.jpg)

选择安装的操作系统

客户机操作系统选择【Apple Mac OS X（M）】，版本我们选【macOS 10.15】

![](https://pic4.zhimg.com/v2-c55043b7347d673b5ac0634504640be7_r.jpg)

这里我们可以” 重命名虚拟机 “，设置好【虚拟机名称】和【安装位置】

![](https://pic3.zhimg.com/v2-1cabdf0461d59c7dcd4e8b4b31f3089e_r.jpg)

处理器配置可以【按需配置】

![](https://pic4.zhimg.com/v2-c39043286a06a11ab41535f9bb9c0c43_r.jpg)

内存配置可以【按需配置】

![](https://pic4.zhimg.com/v2-354d3388aee19978286bec6ff0d3c4df_r.jpg)

网络类型选择【使用桥接网络】

![](https://pic3.zhimg.com/v2-1be2566071bfede03477569b8e760ffe_r.jpg)

控制器类型选择【默认推荐】的就好

![](https://pic2.zhimg.com/v2-07c4f92a6c76c25f74153f42acb47de5_r.jpg)

虚拟磁盘类型也选择【默认推荐】的

![](https://pic1.zhimg.com/v2-c323d37b7d65be520c6d47934438d1c4_r.jpg)

选择【创建新虚拟机硬盘】

![](https://pic4.zhimg.com/v2-a122f38f458731e9fc276e04cfebabd7_r.jpg)

【按需配置】磁盘大小

![](https://pic1.zhimg.com/v2-8f8871bd265e7594cf33604eeac39bb0_r.jpg)

确认无误后，点击【下一步】

![](https://pic1.zhimg.com/v2-f2311bd2e9d4ca11da9d21972e3e3f40_r.jpg)

点击【完成】

![](https://pic1.zhimg.com/v2-898d11bbc9d3ed16f6c6870081ab6094_r.jpg)

点击【CD/DVD（SATA）】或【编辑虚拟机设置】

![](https://pic4.zhimg.com/v2-7f8ef39d21b4d9fb385c0e6e421e7543_b.jpg)

插入【ISO 镜像文件】，选择我们之前下载好的【MacOS Big Sur 的镜像】

![](https://pic4.zhimg.com/v2-bbeb8becdc227c007b3b0867cd9633bb_r.jpg)

MacOS 的虚拟机配置就完成了，理论上是可以直接正常启动进入安装界面的，我在测试写教程的时候没有出现过任何报错，但是之前有小伙伴和我说过有些版本安装会有报错，大概是 VM 和 MacOS 版本的兼容性问题，若是大家在启动 MacOS 系统的时候出现报错了，可以尝试【4、常见问题】中的方法解决。

4、常见问题
------

先试下直接开机，看看能不能启动，若是能正常出现苹果 logo 的图标则跳过【4、报错问题】步骤，直接进入安装 MacOS 步骤，若是出现以下报错，可以尝试下面的解决方法：

  
**问题 Ⅰ：**若出现【vmci.sys 版本不正确】错误提示时

![](https://pic4.zhimg.com/v2-b7129c03243da74bfd4da0f71bf63e3b_b.jpg)

找到刚才创建的虚拟机系统文件路径下的 macOS 10.15.vmx（这个命名是之前创建虚拟机机的时候命名的），用记事本打开编辑 .vmx 文件

![](https://pic3.zhimg.com/v2-ece8a746d1e62bde5a149847524eaa4e_r.jpg)

把【vmci0.present = "TRUE"】改为【vmci0.present = "FALSE"】(不包括这个【】括号) 后保存

![](https://pic3.zhimg.com/v2-441f0ecab403abfda5e3e32ac95c3dbe_r.jpg)

**问题 Ⅱ：**若出现【vcpu-0:VERIFY】报错时，找到刚才创建的虚拟机系统文件路径下的 macOS 10.15.vmx（这个命名是之前创建虚拟机机的时候命名的），用记事本打开编辑 .vmx 文件

![](https://pic1.zhimg.com/v2-97b5fb47e8a6cc7dd05c1534d7f3e4a4_r.jpg)

在 【smc.present = "TRUE" 】后添加【smc.version = "0"】(不包括这个【】括号) 后保存

![](https://pic1.zhimg.com/v2-634fa2e11ab0a3448977359508cddab0_r.jpg)

一般修改或添加完上述出现的问题都可以正常启动 Mac 系统了

5、安装 MacOS Big Sur 11.6
-----------------------

接下来，我们启动 MacOS 吧，但出现了以下苹果 logo 是不是很开心呢

![](https://pic4.zhimg.com/v2-f8d011ea078d03dad93ba8153e3bdecf_r.jpg)

设置 MacOS 的系统语言

![](https://pic2.zhimg.com/v2-f2127d8803c0a90bf6bc92545fcbe011_r.jpg)

我们先不安装，先用【磁盘工具】给磁盘做些调整

![](https://pic4.zhimg.com/v2-0fe1a977c1d3bee36a83bdac8e74ca2b_r.jpg)

先使用【磁盘工具】将硬盘抹掉

![](https://pic3.zhimg.com/v2-5986c0e6da85b4127a618fb3bd4f10da_r.jpg)

填写好名称、格式、方案后，我们点【抹掉】

![](https://pic2.zhimg.com/v2-ced444f0de0817cd4636dbe64e556431_r.jpg)

设置好磁盘之后，我们就可以安装系统了，点击【安装 macOS Big Sur】

![](https://pic4.zhimg.com/v2-b18d3a28046217c88d7b9954e6a41b23_r.jpg)

点击【继续】

![](https://pic1.zhimg.com/v2-5c34a5567bcc726d61f3c595bcd45a34_r.jpg)

点击【同意】协议条款

![](https://pic4.zhimg.com/v2-ca873b4c5e969c18a808f2ed6f9dd95f_r.jpg)

【确认】协议条款

![](https://pic4.zhimg.com/v2-cce23689d038f55141d81de52fae712f_r.jpg)

选择安装的磁盘

![](https://pic2.zhimg.com/v2-8037532676400e4bc5a4cba0ddbd2a55_r.jpg)

正在自动执行安装程序，等待即可

![](https://pic4.zhimg.com/v2-fd09df48956610ff2d8235e6ebbd258f_r.jpg)![](https://pic2.zhimg.com/v2-460be672f57b72ef4825e4d8fc0dacf5_r.jpg)

选择国家或地区

![](https://pic1.zhimg.com/v2-f706b55846de21b86bc6ce7ec0907ae0_r.jpg)

选择语言和输入法

![](https://pic4.zhimg.com/v2-62a261e39f9d8855fb51af75ac9c5fff_r.jpg)

选择辅助功能

![](https://pic3.zhimg.com/v2-93f46a9a54046b6805ac9d957572b922_r.jpg)

继续

![](https://pic3.zhimg.com/v2-eff48ae90c1a0fab95fc68c1e5b00826_r.jpg)

这里是数据迁移，我们是新安装的系统，可以选择左下角的【以后】

![](https://pic4.zhimg.com/v2-c7e5723bbc6f4a0168981641c6be97c3_r.jpg)

Apple ID 登录，我们可以先选择左下角的【稍后设置】

![](https://pic4.zhimg.com/v2-b29ec9ac1a3bb6072892aebef9e836cf_r.jpg)

再次提示是否使用【Apple ID 登录】，依旧选择【跳过】

![](https://pic4.zhimg.com/v2-a7a6b1356de4157900a8250bb27e5e43_r.jpg)

条款与条件点击 “同意”

![](https://pic1.zhimg.com/v2-d6a7008c33573a56d24710495ef3f660_r.jpg)

再次确认【条款与条件】，依旧选择【同意】

![](https://pic3.zhimg.com/v2-784c7eec7ee041164895c9ba74c4c5a2_r.jpg)

设置系统账户名和密码

![](https://pic3.zhimg.com/v2-b5302a9efeb1ba208b6d793f75e8ab66_r.jpg)

快捷设置可以以后再设置，我们依旧继续

![](https://pic1.zhimg.com/v2-c7206233ced203076d06f291849e567c_r.jpg)

使用数据分析和服务，不用勾选，直接继续

![](https://pic2.zhimg.com/v2-bde6469b191bf7dfee33ede6f0582275_r.jpg)

屏幕使用时间可以以后再设置，选择【稍后设置】

![](https://pic2.zhimg.com/v2-7d9d992e6ab8fc420390793083130935_r.jpg)

是否启用语音助手 Siri，根据自己需求选择，我这里选择关闭，之后继续

![](https://pic4.zhimg.com/v2-3d8a77c8ee9813558119c3da8bc84bb3_r.jpg)

外观皮肤选择

![](https://pic1.zhimg.com/v2-1684d12e1afde3da791b6aa6ac9cac84_r.jpg)

设置完成，进入桌面

![](https://pic3.zhimg.com/v2-9e14cdaf695aa768852225d075a6d6da_r.jpg)

进去了是不是很开心呢

![](https://pic1.zhimg.com/v2-d99109d2d49208b4d61e6f76e183d3dc_r.jpg)

6、关于下载
------

6.1 目前支持 OneDrive、百度网盘、阿里云盘、城通网盘、蓝奏云，大家可根据自己习惯自行食用

![](https://pic1.zhimg.com/v2-84f1520e7187a392df5ac9be5dc5756c_r.jpg)

6.2 各大网盘的下载链接都放置 word 文本中，如图【6-2】

![](https://pic1.zhimg.com/v2-2762195a7f9620063517e55ab9bd0324_b.jpg)

6.3 部分网盘由于网盘自身规则限制无法显示或者暂缺等现象，请转至其他网盘下载

6.4 下载速度

> **OneDrive：**可网页直接下载，下载速度 150k~4m 左右，客户端可能会快些，微软账户登录，一般 win11、win10 操作系统自带 OneDrive**_（适合大存储用户适合囤货）_**  
> **百度网盘：**懂得都懂，需要登录百度账号方可下载，非会员速度 100k 左右，会员速度拉满**_（适合会员用户）_**  
> **阿里网盘：**暂时不限速，登录账号下载，超过 100m 文件需要使用客户端下载，暂不支持压缩文件分享**_（适合非压缩资源类型的下载）_**  
> **城通网盘：**可网页直接下载，网页下载速度 100k 左右，客户端 150k 左右，会员速度拉满**_（适合下载小文件又不想登录账号的，ps：会员用户就不打扰了）_**  
> **蓝奏云：**可网页直接不限速下载，目前支持 300m 之类的的文件下载，但是小弟我只能上传 100m 之类的文件，因为我是免费用户开不起会员，哈哈**_（适合下载小文件）_**

在这里我们就安装完了 MacOS Big Sur 11.6 的安装到这里就结束了，另外再说下，使用 VMware 安装 Mac 哪怕配置给的高还是会出现卡断，使体验感不佳。好了，若大家按照这安装教材遇到了其它问题也可以随时和我说，我及时做修改，谢谢大家。