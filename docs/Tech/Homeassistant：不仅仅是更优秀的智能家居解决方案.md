> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/60414)

> 引言在智能家居市场，米家无疑是垄断地位。

引言
--

在智能家居市场，米家无疑是垄断地位。但是米家对于空调控制的优化不佳。首先是每次打开空调都是从 16° 起步，无法记住上次的温度。米家智能中温度风量只能二选一，只有 16° 和高速或者其他温度和自动风速。就在一个月前，一直稳定的 Yeelight 突然抽风，无线开关开始各种不受控。经多次反馈无果，被迫弃坑。你是否也受够了一成不变的米家？厌倦了多个 App 之间的频繁切换？不应该是让你去适应智能家居系统。现在，甚至不需要太多 Linux 基础，几串代码就可以 DIY 最适合自己的智能家居控制系统。

Homeassistant 能做什么
==================

  

最早接触的 Homeassistnt （下简称 HA）是为了把米家设备接入 Homekit 用 Siri 和自动化管理设备。随着 HA 的版本迭代，HA 越来越稳定与个性化。HA 不光是连接 Homekit 和智能家居的桥梁，更是让家居更适应你需求的管理中心。

![](https://cdn.sspai.com/2020/05/10/7a24921cfc4c491defa273381ee14f62.png)

你是否也曾想过：  

*   开关电脑时自动开关台灯和氛围灯。
*   用 Siri 遥控家里所有的智能设备和空调。
*   日历与灯光同步，提醒你的不光是声音，甚至可以是味道。
*   在一块面板上直观地看到家中温湿度、NAS 运行状态、你的明日安排、车辆定位。
*   任何一个开关、设备状态成为智能家居的一环。 

![](https://cdn.sspai.com/2020/05/12/90b97d6f375df3f88328b8a6630b4a6c.jpeg)人工「智能」Siri 无法打开电脑，但可以打开「电恼」

如果你着迷于更智能的家居，更协调统一的管理，那么你一定要试试最新版本的 Homeassistant 。下面，我将用实例演示 Homeassistant 的安装与部署。

安装 Homeassistant
----------------

### 事先准备

  

Nas 、软路由、树莓派、PC，Homeassistant 可以安装在任何机器上。作为智能家居的网关，稳定是关键。本文将使用树莓派安装。

硬件需求：

*   树莓派（推荐 4 代 + 2G）
*   32G（或以上）SD 卡
*   网线

  

首先下载所需 [树莓派系统](https://www.raspberrypi.org/downloads/raspbian/) 并用 [balenaEtcher](https://www.balena.io/etcher/) 刷入。拔出 SD 卡后再插入电脑，在 root 根目录创建新文本文档改名为  `SSH` 。使用终端、 putty 连接树莓派，连接默认密码为 raspberry（输入密码时不显示）。 Win 系统可使用 [finalshell](http://www.hostbuf.com/) 查看树莓派当前网络下载速度。

![](https://cdn.sspai.com/2020/05/10/44e1cedc0426fd51cb946fbfa3edeefb.png)

```
sudo timedatectl set-timezone 'Asia/Shanghai'

```

**设置完时区后重新启动进入下一步。**

### 一键安装

经过开发者和极客用户的不断努力，让普通用户的入门门槛变得越来越低，本文使用 [neroxps](https://bbs.iobroker.cn/t/topic/1404) 制作的一键安装脚本安装。该脚本集成 Docker 安装、更换国内源和 Hassio 安装。

```
sudo -s

```

```
wget https://code.aliyun.com/neroxps/hassio_install/raw/master/install.sh
chmod a+x install.sh
./install.sh

```

回车运行选择自己设备即可。值得注意的是，由于国内网络原因，会出现下载速度其慢或卡死，用 `Ctrl+C` 停止任务后再次运行脚本。此外，也可以在路由端添加代理规则，让下载更加顺畅。

 [官方文档](https://www.home-assistant.io/getting-started/) 的安装方法，优势在于更加稳定。但是受限于国内网络，该安装方法可能会启动失败，且无法安装插件。

安装完成后，便可通过 `http://树莓派IP:8123` 登录 HA 管理界面。

![](https://cdn.sspai.com/2020/05/10/984c8b87c0b15885574a08d01dab1106.png)

配置 Homeassistant
----------------

HA 设备的接入方式并非一键连接，需要在树莓派目录 `\usr\share\hassio\homeassistant` 下**修改配置文件 _configuration.yaml_** 。

进入 HA 管理界面，点击侧边栏 _Supervisor_ 中的 _Add-on store_ ，安装并启动 _File editor_ 插件便可在网页对 _configuration.yaml_ 文件进行直接编写。

![](https://cdn.sspai.com/2020/05/10/02990de6715aec99bd4f9a38bb3171c2.png)

下面将以案例的形式讲解设备的接入。  

### [Yeelight](https://www.home-assistant.io/integrations/yeelight/)

  

以 Yeelight 灯带为例（非原生 Homekit 设备）。HA 自动搜索无法正确配置 Yeelight 灯带，因此需要手动加入。从路由器或 App 中获取**设备的 IP 地址**即可无缝接入 HA。

将以下内容拷贝至 _configuration.yaml_ 中并更改 IP 地址。（注意 _yaml_ 格式和缩进）

```
# 该方法不适用于已支持 Homekit 设备
discovery:
  ignore:
    - yeelight
yeelight:
    devices:
      192.168.1.110:
        name: 灯带 1
      192.168.2.111:
        name: 灯带 2

```

在 _配置_ - _服务器控制_ 重新启动后便可在首页看到设备。

### [Broadlink](https://www.home-assistant.io/integrations/broadlink/) 博联全系列

博联接入 HA 最为省心。以博联智能开关为例，

```
# 「switch:」只需填写一次
switch:
  - platform: broadlink
    host: IP地址 1
    mac: MAC地址 1
  - platform: broadlink
    host: IP地址 2
    mac: MAC地址 2

```

### [XIAOMI](https://www.home-assistant.io/integrations/switch.xiaomi_miio/) 小米系列

区别于其他智能设备，小米设备不光要获取 IP 地址，还要要获取设备 **[API token](https://www.home-assistant.io/integrations/vacuum.xiaomi_miio/#retrieving-the-access-token)**

Token 的获取方法有很多，这里介绍最简单的一种。由 [SchumyHao1](https://bbs.iobroker.cn/t/topic/357) 分享，[下载](https://www.kapiba.ru/search/label/Mi%20Home%20156)  apk 在手机安装。在 app  _通用设置_ - _网络信息_ 中即可看到设备 token 。iOS 用户须下载安卓模拟器运行。

以小米智能开关为例，

```
# 「switch:」只需填写一次
switch:
  - platform: xiaomi_miio
    host: IP 地址
    token: TOKEN 码

```

小米智能网关需用 **key** 连接。还是用到之前的 App , 在网关页面中点击 _关于_ ，多次点击 _插件版本_ 位置打开开发者模式。打开 _局域网通信协议_ 便可获取 _key_ 。

```
xiaomi_aqara:
  discovery_retry: 10
  gateways:
    - key: 获取的密码

```

连接成功后，小米网关下的所有智能硬件将自动接入 HA 。

### [Homekit](https://www.home-assistant.io/integrations/homekit/)

设备连接 HA 后，可利用 HA 内置的 Homekit 插件将所有设备接入家庭，用 iPhone 统一控制。在 _configuration.yaml_ 中写入：

```
homekit:
    exclude_entities:
#以下实体不在 Homekit 中显示 （非必填，格式展示请勿复制）
      - binary_sensor.switch_xxxxxx
      - binary_sensor.wall_switch_xxxxxx
      - ……

```

重启 HA ，在通知中可看到二维码和 8 位连接码，扫描或手动接入即可。原生 Homekit 设备需先在**家庭 App 中移除**， HA 会自动发现，输入 Homekit 设备 8 位连接码后即可接入 HA 。

**更多设备的接入可在 [官方整合文档](https://www.home-assistant.io/integrations/) 中查询。**

至此，Homeassistant 的设备接入工作完成。

Homeassistant 自动化
-----------------

  

接入 HA 的设备，无论是开关、传感器（温湿度等特殊除外）、日历等，一般表示为 `on` 和 `off` 两个状态，HA 监测**设备状态、调用服务**达到家居自动化目的。

HA 前端已集成了自动化配置界面，下面将演示基础的自动化配置流程。

[Google 日历](https://www.home-assistant.io/integrations/calendar.google/) 中有活动时打开灯泡为例，

在 HA 管理界面， _配置_ - _自动化_ 中，点击右下角 + 号创建新自动化。当有活动时，日历状态从 `off` 变为 `on` ，那么将调用 `switch.turn_on` 服务。

![](https://cdn.sspai.com/2020/05/10/82adf2d21ae195fbc0c2d3e7df52df10.png)![](https://cdn.sspai.com/2020/05/10/f397aea6deceea46b3f0f1f9ce21b933.png)

当然，用户也可直接编写 **_automations.yaml_** 文件，以 [小米无线开关](https://www.home-assistant.io/integrations/xiaomi_aqara/#long-press-on-smart-button-1st-generation) 为例，

```
- alias: 工作模式
  trigger:
    platform: event
    event_type: xiaomi_aqara.click  
    event_data:
      entity_id: binary_sensor.switch_xxxxxxxxxxx 
      click_type: single 
  action:
#打开电脑
    - service: switch.turn_on 
      entity_id: switch.mypc
#开灯并调到指定颜色
    - service: light.turn_on 
      data:
        entity_id: light.table_light
        brightness: 255
        rgb_color: [255, 145, 26]
#打开空调并调整到指定温度
    - service: climate.set_temperature
      data:
        entity_id: climate.bedroom
        temperature: 23
        hvac_mode: cool

```

保存后，在 _服务器控制_ 中 _重载自动化_ 

需开启高级模式 即可，无需重启。

![](https://cdn.sspai.com/2020/05/10/f071616ddddb4cb9e11a8ee208a8570a.png)请在用户资料中开启高级模式

自动化保存后会以开关形式在 HA 和 Homekit 中显示，该开关也嵌套在新的自动化中。  

课程表接入 HA ，在上课时开启房间最亮的灯，其效果不亚于上课睡觉时被点名回答问题。以至于我每次只能心惊胆战地睡觉。其次是启动电脑时开启一系列开关，HA 有自带集成 [wake on lan](https://www.home-assistant.io/integrations/wake_on_lan/) 可 Ping 电脑，但是我总觉得不太安稳。我这里使用的是魔改的小米门窗传感器检测电脑电源状态。

![](https://cdn.sspai.com/2020/05/10/3c1e2a2310e0ceb3fdfaa4a8a5a6614b.jpg)

Homeassistant 进阶
----------------

截至 0.109 版本，HA 可接入设备已经超过 1500 种。 PC 、 Synology 、特斯拉等都可以通过 HA 的内置插件直接连接。但是其他用户开发的自定义插件、主题、卡片还需手动添加。下面将举例演示如何在 HA 中安装自定义插件。

### 树莓派 Samba 安装与部署

安装 samba

```
sudo apt-get update
sudo apt-get install samba samba-common-bin

```

配置 samba

```
sudo nano /etc/samba/smb.conf

```

将以下内容（注意格式）添加到文件最下方，`Ctrl+X` 退出并 `Y` 保存文件。

```
[Hass]
    comment = Homeassistant
    valid users = pi,root
    path = /usr/share/hassio
    browseable = yes
    writable = yes

```

重启 samba 服务

```
sudo samba restart

```

添加登录账户并创建密码。

```
sudo smbpasswd -a pi

```

修改文件权限

```
sudo chmod 777 -R /usr/share/hassio

```

在访达或此电脑中输入 `//树莓派IP` 以账户名 `pi` 访问 `\hass\homeassistant` 配置文件。

### 插件安装  

由于 HA 没有内置红外码库，用户想要遥控空调，只能自行学码或者安装插件。

下面演示 [SmartIR](https://github.com/smartHomeHub/SmartIR) 空调遥控插件的安装流程。

*   `\homeassistant` 下 **创建新文件夹 `custom_components`**
*   [下载](https://github.com/smartHomeHub/SmartIR/archive/master.zip)[](https://github.com/smartHomeHub/SmartIR/archive/master.zip) 插件，将压缩包中 `smartir` 文件夹拖入`\homeassistant\custom_components` 中 。

![](https://cdn.sspai.com/2020/05/10/4af89de6fafc5f521d5b28dc2d3f5dac.png)custom_components 文件夹需自行创建

*   在 _configuration.yaml_ 中写入：

```
#博联红外遥控器
smartir:
switch:
  - platform: broadlink
    host: 192.168.10.10          
    mac: '00:00:00:00:00:00'     
climate:
  - platform: smartir
    name: Office AC               
    unique_id: office_ac          
    device_code: 1000       #参照插件目录获取空调代号
    controller_data: 192.168.10.10       #博联RM IP地址
    temperature_sensor: sensor.temperature  #温湿度传感器在 HA 中的ID名
    humidity_sensor: sensor.humidity
    power_sensor: binary_sensor.ac_power

```

重启 HA 后可见空调控制面板。

**更多插件可访问 [官方论坛](https://community.home-assistant.io/) 。**

### 官方插件市场 HACS

Add-store 是功能性插件下载中心， [HACS](https://hacs.xyz/) 则是自定义 UI 的下载中心。

HACS 本质上是插件，安装方式同上方。 [下载](https://github.com/hacs/integration/releases/tag/0.24.3) 解压后将整个 `hacs` 文件夹拖入 `custom_components` 后重启即可完成安装。

之后在 _配置_ 右下角 + 号中搜索 「hacs」。

![](https://cdn.sspai.com/2020/05/10/a662e39d98b5118395a764be3f652350.png)

Github 个人访问令牌请从 [这里](https://github.com/settings/tokens) 创建。无需勾选其他选项，创建完成后复制粘贴即可完成配置。

HACS 中包含了用户制作的卡片样式，主题等，可一键安装。主题安装后，可在用户资料中更改。卡片需自行配置。

下面以 [button-card](https://github.com/custom-cards/button-card) 为例，在 HACS _PLUGINS_ 中选择自己想要的卡片下载安装后，点击 **ADD TO LOVELACE** 按钮。

![](https://cdn.sspai.com/2020/05/10/b035c48fc095dd5fb5b3972a0fd95d62.png)

在  _概览_ 右上角  _配置 UI_ 中添加 _水平堆叠_ 卡片，将卡片依次加入。

![](https://cdn.sspai.com/2020/05/10/1263bf9792a1d2930e436f37cea049db.png)

```
type: 'custom:button-card'   #卡片类型（必填）
entity: switch.tai_deng  #设备ID （必填）
icon: 'mdi:desk-lamp'  #图标
size: null  #根据需要配置
styles:  
  name:
    - font-size: 13px
    - align-self: middle

```

官方提供了部分图标，可在 [这里](https://materialdesignicons.com/) 选取。用户也可将自行制作的图标放在  `\homeassistant\www` 下，用 `entity_picture: /local/xxx.png` 调取。**推荐在 _配置_ - _自定义_ 中进行全局更改**。  

Homeassistant × 屏幕
------------------

树莓派上连接一块触屏，登陆 HA 管理界面，就可以直接控制家里的智能设备。

但这个方法有许多不足。大多小尺寸触摸屏（10 寸以下）非独立供电，无法自动化熄屏。DIY 的小屏幕虽然可以通过连接智能开关熄屏，但是触控质量参差不齐，不建议大家踩坑。秉持「万物都应自动化」原则，我淘了一块亚马逊平板，1920*1200 的分辨率，价值 300 元。大家也可以选择手边的闲置安卓平板。

HA 提供 App ，需在 Google Play 上下载，若无此条件也可直接使用浏览器、将网页打包成 app 或使用 [APKPure](https://apkpure.com/cn/) 下载替代 App。值得注意的是，老旧安卓系统内**「WebView」版本过低** 时会导致自定义 UI 显示错误。需在 Google Play 上更新 WebView 版本。

安卓系统开发者模式中有「充电不关闭屏幕」选项，但是依旧会降低屏幕亮度。如有需要可下载 app Tasker ，创建**充电常量，断电熄屏**任务。将充电线插在智能开关上，创建自动化便可在指定时间，或随智能灯泡开关屏幕。

![](https://cdn.sspai.com/2020/05/10/0c30c778a122079721e9e588daf71e17.jpg)

我将只把最常用的开关放在了首页，其他是天气、温度、课程表、汽车状态等信息，尽量做到简单。  

![](https://cdn.sspai.com/2020/05/10/195ef475d13d2243b1d9e54812c9c843.png)

至于为什么要把车辆保养状态这种没用的信息放进去，仅仅是我觉得「把汽车接入智能家居」这件事太酷了。

最后，[官方文档](https://www.home-assistant.io/)、[官方论坛](https://community.home-assistant.io/)、[瀚思彼岸](https://bbs.hassbian.com/forum.php)、[ioBroker](https://bbs.iobroker.cn/) 将会帮到你很多。