---
title: "游戏加速方案（一）：WireGuard"
tags: ["code"]
categories: ["游戏加速方案"]
date: 2019-07-30T13:44:57+08:00
lastmod: 2019-07-30T13:44:57+08:00
draft: true
---

## 简介

最近空闲下来了稍微研究了一下游戏加速器方面的东西，稍显繁杂。主要源于以下几点：

1. 游戏连接到服务器大部分都是透过udp连接，部分还需要NAT穿透
2. 但是主流的代理程序都是基于tcp的
3. 最后运营商存在udp通道限速问题

V2ray在UDP支持太糟糕，它对游戏加速的效果并不是很好（具体可以参考一下github的issue，不展开说了）。所以大家在搭建透明代理的时候，可以利用iptables仅让tcp流量经过v2ray，然后udp流量则经过我们今天的主角：WireGuard

## WireGuard

### 它是什么？

WireGuard可以说是下一代代理程序。从技术层面来说，安全性极高（甚至能够抵御未来的量子攻击）、全平台都有客户端（包括openwrt）、基于Linux内核运行、效率极高，速度很快！缺点嘛，它目前还比较新，应用不是非常广泛，另外我的体验下来感觉在分流策略方面做的不是很好。

### 安装

我用的系统是Ubuntu18.04，若是其他系统那么在安装方面会有所差异。

```bash
# 首先需要安装匹配内核的linux-headers
apt update
apt install linux-headers-$(uname -r) -y
# 安装WireGuard，非ubuntu系统参看官网
add-apt-repository ppa:wireguard/wireguard
apt update
apt install wireguard resolvconf
```

运行wg，要是没有任何错误和输出，那就是安装完成了。

### 配置服务端

首先要完成一些准备工作，WireGuard基于Linux内核级运行，所以它需要创建虚拟网卡。

```bash
# 开启ipv4流量转发
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
# 生成服务端密钥对
cd /etc/wireguard
wg genkey | tee server_privatekey | wg pubkey > server_publickey
# 生成客户端密钥对
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```

接着就是编写服务端的配置文件了，我们需要用到服务端/客户端密钥对。以下是范例，大家自己用喜欢的编辑器写入/etc/wireguard/wg0.conf即可。

```conf
[Interface]
  PrivateKey = server_privatekey
  Address = 10.0.0.1/24
  PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
  ListenPort = 51820
  DNS = 8.8.8.8
  MTU = 1200

[Peer]
  PublicKey = client_publickey
  AllowedIPs = 10.0.0.2/32
```

### 编写客户端配置文件

注意！虽然我们的标题是生成客户端配置文件，但是这些操作都是在服务端完成的！以下是范例，大家用自己喜欢的编辑器写入/etc/wireguard/client0.conf即可。

```conf
[Interface]
  PrivateKey = client_privatekey
  Address = 10.0.0.2/24
  DNS = 8.8.8.8
  MTU = 1200

[Peer]
  PublicKey = server_publickey
  Endpoint = server_ip:51820
  AllowedIPs = 0.0.0.0/0, ::0/0
  PersistentKeepalive = 25
```

### 启动服务端并设置开机自启

```bash
# 挂起配置好的虚拟网卡
wg-quick up wg0
# 查看运行状态，若没有错误则设置开机自启
wg && systemctl enable wg-quick@wg0
```

### 生成客户端二维码

```bash
# 安装qrencode
apt install qrencode
qrencode -t utf8 < /etc/wireguard/client0.conf
```

