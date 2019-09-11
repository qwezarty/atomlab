---
title: "浙江大学校内有线网和树莓派路由器"
tags: ["code"]
categories: ["树莓派"]
date: 2019-09-10T19:19:35+08:00
lastmod: 2019-09-10T19:19:35+08:00
author: "qwezarty"
draft: false
---

## 简介

虽然对于校园网不能要求太多，不过依然要吐槽一下浙江省的校园网，连我浙都不能够幸免。万幸的是，玉泉校区并没有采用电信的闪讯方案（啊啊啊！），而是采用了标准的l2tp代理服务，故我等linux用户得以救赎。其实作为国内最高学府之一，linux用户要是不能够上网那也太可笑了。

进入正题，本文并不是step-by-step的教程（所以不要看到命令行就copy/paste），所以也不会做多发行版兼容，但作为综述并旨在讲解一些整体细节和遇到的坑，可以帮你节省不少的时间。我会假设你拥有linux的系统基础，并以debian系为例讲解，大概率适用于macos用户（cheer up!）

在最后有线网络连接上以后，还会将树莓派（pi3b+）中的无线网卡改为AP模式，散出热点供多设备使用。

大家都有自己遇到的问题，如果Google后无解那么当然可以邮件给作者我，你可以在本站找到我的邮箱，只要你是ZJUer我会尽力帮助你。

## 有线连接

### 安装xl2tpd

首先更新软件包列表，仅想要连接有线的同学需要依赖：xl2tpd，其实这部分在信息网上能够找到linux的配置手册。需要注意，本文均略去了sudo，若出现权限问题大家酌情加上即可。

```bash
# 更新软件包列表
apt update
# 安装xl2tpd
apt install xl2tpd -y
```

这里着重说一下树莓派如何在有线网卡离线的情况下安装，我采用的方法是手机共享出热点后用pi3b+的内置无线网卡连接。若是低于此版本的pi，那么你只能采用离线依赖包手动安装了。无线配置方法有两种，如下：

1. 使用raspi-config自带的network选项连接（推荐）
2. 手动通过配置或者命令行来连接，注意iw是不能够连接wpa/psk2类型加密的无线网络的，所以我们要使用wpa_supplicant命令来连接到手机热点，你可以从google中搜索到配置如何写

### 编写配置文件

需要修改的配置文件比较多，内容就直接贴出了，不做说明，它们有：

1. /etc/xl2tpd/xl2tpd.conf，主配置文件，仅在更改账户时需重新修改
2. /etc/ppp/options.xl2tpd.zju，模拟拨号的配置，基本一次配置后无需修改
3. /etc/ppp/chap-secrets，VPN拨号时用的账户密码，仅在更改账户或者密码时需修改

以下是主配置文件，位于/etc/xl2tpd/xl2tpd.conf。其中，USERNAME@TYPE是你的学号@类型，类型有a, c, d，分别对应10/30/50的套餐。

```conf
[lac ZJU_VPN]							; Example VPN LAC definition
lns = 10.5.1.7
redial = yes							; * Redial if disconnected?
redial timeout = 15					; * Wait n seconds between redials
max redials = 5						; * Give up after n consecutive failures
require pap = no						; * Require PAP auth. by peer
require chap = yes					; * Require CHAP auth. by peer
require authentication = yes			; * Require peer to authenticate
name = USERNAME@TYPE							; * Report this as our hostname
ppp debug = yes						; * Turn on PPP debugging
pppoptfile = /etc/ppp/options.xl2tpd.zju	; * ppp options file for this lac
```

以下是模拟拨号的配置，位于/etc/ppp/options.xl2tpd.zju

```conf
noauth
proxyarp
defaultroute
```

以下是账户密码保存的位置，位于/etc/ppp/chap-secrets。注意，中间用tab隔开就好，那个是星号你没看错，PASSWORD记得加上双引号（虽然很可能不加也没事，不过我还是配置手册里这样写我就照做了）

```conf
USERNAME@TYPE	*	"PASSWORD"	*
```

### 启动VPN拨号

如同文档所写的一样，我们需要启动服务后然后开始拨号，成功后会生成一张ppp0的虚拟网卡。我们需要记住这个ppp0的inet地址以供后续使用。

但是文档没告诉你的一点就是，你需要先在宿管中心公众号申请一个固定ip（用mac地址），然后再修改有线网卡（本文以eth0为例）变为固定ip，我们需要首先改动/etc/dhcpcd.conf，追加以下内容：

```conf
interface eth0
# /24代表的是子网掩码cidr表示法，代表255.255.255.0
static ip_address=10.xxx.xx.xxx/24
static routers=10.xxx.xx.x
static domain_name_servers=10.xxx.xx.x
```

在这一切配置完成后，你需要重启networking和dhcpd服务，也可以重启（反正我是重启了）来使静态ip生效。

```bash
# 启动服务
systemctl start xl2tpd
# 注意查看status是否是running
systemctl status -l xl2tpd
# 开始拨号
echo 'c ZJU_VPN' > /var/run/xl2tpd/l2tp-control
# 查看ppp0的inet地址
ip addr show ppp0
```

这里再做一些补充，首先是每次拨号后ppp0的ipv4地址都会更换！然后，如果你的网卡是支持ipv6（pi3b+是支持的）的话，那么走的确实是ipv6的通道，我测试过对于某些网站支持的并不良好，所以你可能需要修改内核参数来关闭它（可选）。最后，若是为自动化考虑，你可以使用如下命令来抑制其他垃圾输出：

```bash
# 全局地关闭ipv6的功能（可选，ipv6延迟确实要低一半但部分网站不兼容）
echo "net.ipv6.conf.all.disable_ipv6=1" > /etc/sysctl.conf
sysctl -p
# 仅输出ppp0的inet地址
ip -4 addr show ppp0 | grep inet | awk '{print $2}'
```

最后一步就是修改路由，这里我建议大家写成一个简单的shell脚本，不然每次断开或者重启后都要查看ppp0的inet地址，太过于繁琐。为方便考量，同时我把拨号也集成进去了，大家以做参考。

```bash
# 拨号，产生ppp0的interface
echo 'c ZJU_VPN' > /var/run/xl2tpd/l2tp-control
# 睡几秒钟，等拨号产生ppp0
sleep 5
# 注意网关要修改成你申请时拿到的那个
DEF_GW=10.110.33.1
VPN_SERV=10.5.1.7
VPN_GW=$(ip -4 addr show ppp0 | grep inet | awk '{print $2}')

# 修改路由表
route add -host $VPN_SERV gw $DEF_GW metric 1 dev eth0
route add –net target netmask NM gw $DEF_GW metric 1 dev eth0
route del –net default gw $DEF_GW
route add –net default gw $VPN_GW metric 1 dev ppp0
```

将它保存成.sh文件后，使用bash执行即可，至此顺利连接上有线网络（可以用curl ip.gs来查看是否连通）

## 无线热点

本节的内容主要是使用hostapd将无线网卡改成AP模式，并通过iptables搭建nat网络。我们主要是会依赖两个包，分别是：hostapd, dnsmasq。另外假定无线网卡的interface为wlan0，ip段则使用192.168.0.x

安装就不多说了，需要修改三个地方（都是追加），依次是：

/etc/dnsmasq.conf

```conf
interface=wlan0
dhcp-range=192.168.0.11,192.168.0.30,255.255.255.0,24h
```

/etc/dhcpcd.conf

```conf
interface wlan0
static ip_address=192.168.0.10/24
```

/etc/hostapd/hostapd.conf

```conf
# 以下所有配置都是必须的
interface=wlan0
ssid=your_ssid
# 注意我使用了5GHZ的频段
# 2.4ghz是g
hw_mode=a
# 2.4ghz设置为7
channel=48
wmm_enabled=0
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=your_password
wpa_key_mgmt=WPA-PSK
```

最后一步！非常简单的NAT组网，请大家手动增加到shell脚本中即可。

```bash
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE
```

OK，至此所有配置写完，请大家把dnsmasq/hostapd/dhcpd/xl2tpd等服务设置为开机自启动。当在系统重启后，可以仅仅运行上面我提供给你们的脚本即可一键上网，有线和热点能够全通。

## 其他技术细节

### 允许ssh连接

在树莓派上允许ssh连接非常的简单，你可以通过raspi-config中的interfacing options/ssh选项卡来修改。

### 关于inet6

其实我成功设置了ipv6，但是不知道为什么网络性能的表现反而下降了，这块内容会在未来逛逛cc98以后再尝试成功以后进行补充。

## 参考资料

1. 主要的参考资料其实是官方提供的linux用户连接手册
2. 其次是后来发现的cc98上的帖子，[浙大紫金港宿舍Linux上网详解](https://m.cc98.org/topic/3938990)

