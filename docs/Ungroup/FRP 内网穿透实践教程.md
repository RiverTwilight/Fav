> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/337939762)

什么是内网穿透
-------

内网穿透，也即 NAT 穿透，进行 NAT 穿透是为了使具有某一个特定源 IP 地址和源端口号的数据包不被 NAT 设备屏蔽而正确路由到内网主机。 说人话就是我们想在不连接家里的 wifi 的情况下访问我们家里面的电脑和其他设备时，由于家庭宽带没有固定的公网 ip 使得我们无法向访问云服务器一样 直接访问，这时候我们就需要使用到内网穿透技术，让我们在其他网络下也能访问到处于内网环境的设备。而内网穿透的工具也有很多如： - 花生壳 - nat123 - ngrok - frp

本文主要讲 FRP 的搭建和使用。

什么是 FRP
-------

Frp (Fast Reverse Proxy) 是比较流行的一款。FRP 是一个免费开源的用于内网穿透的反向代理应用，它支持 TCP、UDP 协议， 也为 http 和 https 协议提供了额外的支持。你可以粗略理解它是一个中转站， 帮你实现 公网 ←→ FRP(服务器) ←→ 内网 的连接，让内网里的设备也可以被公网访问到。

![](https://pic2.zhimg.com/v2-a54037f04aaf8d33a870a70a80a1d0c5_r.jpg)

而目前 FRP 还推出了 “点对点穿透” 的试验性功能，连接成功后可以让公网设备直接跟内网设备 “点对点” 传输，数据流不再经过 VPS 中转， 这样可以不受服务器带宽的限制，传输大文件会更快更稳定。当然，此功能并不能保证在你的网络环境 100% 可用，而且还要求访问端也得运行 FRP 客户端 (因此目前手机是无法实现的，只有电脑可以)。由于实现条件较多，所以有文件传输需求的朋友还是建议买带宽稍大一点的 VPS 会比较省心。 [仓库地址](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp)

安装 FRP
------

首先我们得要有一台有公网 ip 的服务器，比如各大云服务器。然后我们就需要去服务器上安装服务端。

### 服务端安装与配置

FRP 使用 Go 语言开发，可以支持 Windows、Linux、macOS、ARM 等多平台部署。FRP 安装非常容易，只需下载对应系统平台的软件包并解压就可用了。 这里以 Linux 系统 Ubuntu 18.04 为例：

*   首先下载对应的安装包。自行选择对应的版本 [https://github.com/fatedier/frp/releases](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases)

```
curl -# -LJO https://github.com/fatedier/frp/releases/download/v0.34.3/frp_0.34.3_linux_amd64.tar.gz

```

没有安装 curl 要先安装 curl 当然 wget 同样也能下载

```
wget https://github.com/fatedier/frp/releases/download/v0.34.3/frp_0.34.3_linux_amd64.tar.gz

```

*   使用 tar 指令解压 tar.gz 文件 `tar -zxvf frp_0.34.3_linux_amd64.tar.gz`  
    
*   进入解压后的文件夹

![](https://pic2.zhimg.com/v2-e9d2b103f8840e79682e0ae222053cc1_r.jpg)

由于我们是服务端所以只需要关注 frps 相关文件就行了 - 接下来我们开始对服务端进行配置 `vim frps.ini`

```
[common]
bind_port = 7000
bind_addr = 0.0.0.0
token = 123456

dashboard_port = 37500
dashboard_user = admin
dashboard_pwd = admin

```

> [common]部分是必须有的配置, 其中 bind_port 是自己设定的 frp 服务端端口，bind_addr 是绑定的 ip 地址默认为 0.0.0.0 即本机的所有 ip 地址。 token 用于验证连接，只有服务端和客户端 token 相同的时候才能正常访问。如果不使用 token，那么所有人都可以直接连接上，所以建议大家在使用的时候还是把 token 加上。 而下面的 [dashboard] 仪表盘的配置（可以不配置） 配置了的化可以访问服务器 ip:dashboard_port 通过 dashboard_user 和 dashboard_pwd 登陆后查看 frp 服务器状态

![](https://pic2.zhimg.com/v2-f50a3dc0730c57fcd932dc2b16d239f1_r.jpg)

服务端这样配置就可以了，其他高级功能请参考[官方文档](https://link.zhihu.com/?target=https%3A//gofrp.org/docs/)。 - 启动服务端 `./frps -c frps.ini` 当然我们也可以把 frps 注册成系统服务，避免每次重启系统都要去手动启动。 `sudo vim /lib/systemd/system/frps.service` 然后在 frps.service 文件里写入：  

```
[Unit]
Description=frp server
After=network.target

[Service]
Type=simple

ExecStart=/your path/frps -c /your path/frps.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target

```

然后就可以启动 frps 了 `sudo systemctl start frps` 打开自启动 `sudo systemctl enable frps` 查看状态和日志信息： `sudo systemctl status frps`

```
➜  ~ systemctl status frps                    
● frps.service - fraps service
   Loaded: loaded (/lib/systemd/system/frps.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-12-17 17:14:06 CST; 49s ago
 Main PID: 28974 (frps)
    Tasks: 5 (limit: 4465)
   CGroup: /system.slice/frps.service
           └─28974 /root/frp/frps -c /root/frp/frps.ini

Dec 17 17:14:06 VM-0-15-ubuntu systemd[1]: Started fraps service.
Dec 17 17:14:06 VM-0-15-ubuntu frps[28974]: 2020/12/17 17:14:06 [I] [service.go:190] frps tcp listen on 0.0.0.0:700
Dec 17 17:14:06 VM-0-15-ubuntu frps[28974]: 2020/12/17 17:14:06 [I] [service.go:289] Dashboard listen on 0.0.0.0:37
Dec 17 17:14:06 VM-0-15-ubuntu frps[28974]: 2020/12/17 17:14:06 [I] [root.go:215] start frps success

```

如果要重启应用，可以这样，`sudo systemctl restart frps` 如果要停止应用，可以输入，`sudo systemctl stop frps` 到这里服务端已经配置完成，你已经可以访问你的 frp 仪表盘了。

### 客户端的配置

*   首先客户端也要先去下载对应的安装包。自行选择对应的版本 [https://github.com/fatedier/frp/releases](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases)
*   然后我们需要对 frpc.ini 进行配置 这里以 windows 远程文件共享为例

```
[common]
# 服务器的公网地址
server_addr = X.X.X.X
# 7000为服务端frp与客户端frp相互通信的端口就是我们服务端配置的监听端口
server_port = 7000
token = 123456

[smb]
# win10文件共享smb协议通过tcp通信
type = tcp
local_ip = 127.0.0.1
# smb协议的本地端口
local_port = 445
# 设定远程端口，当访问服务器的7002端口时，数据会被转发到本地445端口
remote_port = 7002

```

其他配置类似，请参考[官方文档](https://link.zhihu.com/?target=https%3A//gofrp.org/docs/)。 - 接着我们就可以启动客户端了。 powershell 启动客户端 frp `./frpc -c frpc.ini`

> 使用具有访问 smb 服务器能力的软件进行访问。我是用 ios 的 FileExplorer 和 nPlayer 进行访问的。软件设置主机地址为 X.X.X.X，端口为 7002。注意，有些软件不能设置 smb 服务器的端口，比如 OPlayer，它只能使用默认的 445 端口，所以要把 remote_port 配置成 445 才可以使用。  

当然我们还可以设置客户端的自启动，以 Windows 电脑为例： 1. 首先编写启动脚本

```
Set ws = CreateObject("Wscript.Shell")
ws.run "cmd /c c:\frps\frpc.exe -c c:\frps\frpc.ini",vbhide

```

`/c c:\frps`请自行更换成你自己电脑的 frp 所在路径 2. 然后把 vbs 放到启动目录即可 `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`

### 下面是 frp github 介绍中一些配置示例

### ssh

修改 frps.ini：

```
# frps.ini 
[common] 
bind_port = 7000

```

启动 frps：`./frps -c ./frps.ini` 修改 frpc.ini，server_addr 是你的 frps 的服务器 IP：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[ssh] 
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

```

启动 frpcz:`./frpc -c ./frpc.ini` 假设用户名为 test，则通过 ssh 连接到 LAN 中的服务器：`ssh -oPort=6000 test@x.x.x.x`

### 通过自定义域访问 LAN 中的 Web 服务

有时我们希望将 NAT 网络后面的本地 Web 服务公开给其他人以使用您自己的域名进行测试，但是无法将域名解析为本地 IP。我们可以使用 frp 公开 http 或 https 服务。

修改 frps.ini，配置 http 端口 8080：

```
# frps.ini 
[common] 
bind_port = 7000
vhost_http_port = 8080

```

启动 frps：`./frps -c ./frps.ini` 修改 frpc.ini 并将远程 frps 服务器的 IP 设置为 xxxx 这 local_port 是您的 Web 服务的端口：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[web] 
type = http
local_port = 80
custom_domains = www.yourdomain.com

```

启动 frpc：`./frpc -c ./frpc.ini` 解析域名到 frp 的 IP 地址，然后使用 url 访问您的本地 Web 服务 [http://www.yourdomain.com:8080](https://link.zhihu.com/?target=http%3A//www.yourdomain.com%3A8080)。

### 转发 DNS 查询请求

修改 frps.ini：

```
# frps.ini 
[common] 
bind_port = 7000

```

修改 frpc.ini，将远程 frps 的服务器 IP 设置为 xxxx，将 dns 查询请求转发到 google dns 服务器 8.8.8.8:53：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[dns] 
type = udp
local_ip = 8.8.8.8
local_port = 53
remote_port = 6000

```

通过 dig 发送 dns 查询请求：

```
dig @x.x.x.x -p 6000 www.google.com

```

### 转发 unix 域套接字, 使用 tcp 端口连接 unix 域套接字，如 docker 守护进程。

配置与上面相同的 frps。 配置 frpc.ini

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[unix_domain_socket] 
type = tcp
remote_port = 6000
plugin = unix_domain_socket
plugin_unix_path = /var/run/docker.sock

```

通过 curl 命令获取 docker 版本：`curl http://x.x.x.x:6000/version`

### 公开一个简单的 http 文件服务器

配置与上面相同的 frps 配置启动 frpc：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[test_static_file] 
type = tcp
remote_port = 6000
plugin = static_file
plugin_local_path = /tmp/file
plugin_strip_prefix = static
plugin_http_user = abc
plugin_http_passwd = abc

```

访问`http://x.x.x.x:6000/static/`, 输入正确的用户和密码，就可以查看文件`/tmp/file`。

### 在安全性中公开您的服务

对于某些服务，如果直接将它们暴露给公共网络将存在安全风险。stcp（secret tcp）帮助您创建代理，避免任何人可以访问它。 配置与上面相同的 frps。 启动 frpc，转发 ssh 端口并且 remote_port 没用：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[secret_ssh] 
type = stcp
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22

```

启动另一个要连接此 ssh 服务器的 frpc：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[secret_ssh_visitor] 
type = stcp
role = visitor
server_name = secret_ssh
sk = abcdefg
bind_addr = 127.0.0.1
bind_port = 6000

```

假设用户名为 test，则通过 ssh 连接到 LAN 中的服务器： `ssh -oPort=6000 test@127.0.0.1`

### P2P 模式

xtcp 旨在直接在两个客户端之间传输大量数据。现在它无法穿透所有类型的 NAT 设备。如果 xtcp 不起作用，您可以尝试使用 stcp。

配置 xtcp 的 udp 端口：

```
bind_udp_port = 7001

```

启动 frpc，转发 ssh 端口并且 remote_port 没用：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[p2p_ssh] 
type = xtcp
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22

```

启动另一个要连接此 ssh 服务器的 frpc：

```
# frpc.ini 
[common] 
server_addr = xxxx
server_port = 7000

[p2p_ssh_visitor] 
type = xtcp
role = visitor
server_name = p2p_ssh
sk = abcdefg
bind_addr = 127.0.0.1
bind_port = 6000

```

假设用户名为 test，则通过 ssh 连接到 LAN 中的服务器： `ssh -oPort=6000 test@127.0.0.1`

### 认证

1.  从 v0.10.0 开始，您只需要 token 在 frps.ini 和 frpc.ini 中进行设置。
2.  请注意，frpc 和 frps 服务器之间的持续时间不得超过 15 分钟，因为时间戳用于身份验证。然后，可以通过设置 authentication_timeoutfrps 的配置文件来修改此超时持续时间。它的 defalut 值是 900，意味着 15 分钟。如果它等于 0，则 frps 将不检查身份验证超时。

### 端口重用

现在 vhost_http_port 和 vhost_https_portfrps 可以使用相同的端口 bind_port。frps 将检测连接的协议并相应地处理它。 我们希望尝试允许多个代理在将来使用不同的协议绑定相同的远程端口。

### 支持 KCP 协议

frp 支持 kcp 协议，自 v0.12.0 起。 KCP 是一种快速可靠的协议，可以实现将平均延迟降低 30％至 40％并将最大延迟降低三倍的传输效果，其代价是浪费 10％至 20％的带宽浪费比 TCP。 在 frp 中使用 kcp：

```
#frps.ini 
[command] 
bind_port = 7000
kcp_bind_port = 7000 # KCP需要绑定一个UDP端口

```

配置 frpc 中使用的协议连接 frps：

```
# frpc.ini 
[command] 
SERVER_ADDR = XXXX
SERVER_PORT = 7000
protocol = KCP

```

### 负载均衡

支持负载均衡 group。此功能仅适用于 tcp 现在的类型。

```
# frpc.ini 
[test1] 
type = tcp
local_port = 8080
remote_port = 80
group = web
group_key = 123

[test2] 
type = tcp
local_port = 8081
remote_port = 80
group = web
group_key = 123

```

同一组中的代理将随机接受来自端口 80 的连接。

### 重写主机标头

转发到本地端口时，frp 根本不会修改隧道 HTTP 请求，它们会在收到时逐字节地复制到服务器。某些应用程序服务器使用 Host 标头来确定要显示的开发站点。因此，frp 可以使用修改后的主机头重写您的请求。使用该 host_header_rewrite 开关重写传入的 HTTP 请求。

```
# frpc.ini 
[web] 
type = http
local_port = 80
custom_domains = test.yourdomain.com
host_header_rewrite = dev.yourdomain.com

```

如果 host_header_rewrite 指定，则将重写主机头以匹配转发地址的主机名部分。

### 在 HTTP 请求中设置标头

您可以为代理类型设置标头 http。

```
# frpc.ini 
[web] 
type = http
local_port = 80
custom_domains = test.yourdomain.com
host_header_rewrite = dev.yourdomain.com
header_X-From-Where = frp

```

请注意，具有前缀的参数 header_将添加到 http 请求标头中。在此示例中，它将标头设置 X-From-Where: frp 为 http 请求。

### 获取客户端真正的 IP

仅限 http 代理的功能。 你可以从 HTTP 请求头获取用户的真实 IP X-Forwarded-For 和 X-Real-IP。请注意，现在您只能在每个用户连接的第一个请求中获取这两个标头。

### URL 路由

frp 支持通过 url 路由将 http 请求转发到不同的后向 Web 服务。 locations 指定用于路由的 URL 前缀。frps 首先搜索由文字字符串给出的最具体的前缀位置，而不管列出的顺序如何。

```
# frpc.ini 
[web01] 
type = http
local_port = 80
custom_domains = web.yourdomain.com
locations = /

[web02] 
type = http
local_port = 81
custom_domains = web.yourdomain.com
locations = / news，/ about

```

HTTP 与 URL 前缀请求 / news 和 / about 将被转发到 web02 和他人 WEB01。

### 范围端口映射

代理名称前缀 range: 将支持映射范围端口。

```
# frpc.ini 
[range：test_tcp] 
type = tcp
local_ip = 127.0.0.1
local_port = 6000-6006,6007
remote_port = 6000-6006,6007

```

frpc 将生成 8 个代理 test_tcp_0, test_tcp_1 … test_tcp_7。

### 插件

frpc 默认只向本地 tcp 或 udp 端口转发请求。而插件提供了丰富的功能。内置的插件有 unix_domain_socket，http_proxy，socks5，static_file。

指定 plugin 参数使用的插件。插件的配置参数应该以 plugin_。local_ip 并且 local_port 对插件没用。