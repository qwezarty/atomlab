---
title: "博客搭建指南（二）：代理服务器"
tags: ["code"]
categories: ["博客搭建指南"]
date: 2018-07-23T15:56:43+08:00
draft: false
---

## 简介

### 这篇文章能带给你什么

* Project V + websocket + tls
* 利用caddy开放某个路由，通过websocket将流量转发到V2ray中
* 文章已经在第一版（2018）的基础上经过多次修订，现在配置更为简单（2020）
* 归档部分若无特殊需要请跳过，在未来我会移除它（暂时保留以供有些同学升级时作为参考）

### 在开始之前，请确保

* 已经读过、或大致浏览过[博客系统构建（一）](/post/ladder/build-blog/)
* 有基本的Linux系统知识
* 能够使用vim/nano/emacs编辑文件
* 更新软件包和列表，到最新版本
* 文章中的部分命令略去了sudo

## Project V

### 什么是V？

[Project V](https://www.v2ray.com/)是一款用Go语言编写的新兴的网络分发、代理、混淆的自由软件。相比较老前辈SS，其配置更加丰富，能够利用它实现的功能会更加多，而对websocket/http2协议的支持，让它的穿透和抗检测能力大幅提高。

*注1：本文的内容基本没有大幅度改动，另外据说现在SS被查的很严（2020）*  
*注2：安装和更新自 Aug 2020 起，有略微变化，你可以在这里找到更多的信息 [fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray)*

### 安装V

``` bash
# 首先安装依赖库
sudo apt update
sudo apt install curl unzip daemon
# 老版本安装方式（弃用）：curl -Ls https://install.direct/go.sh | sudo bash
# 下载安装脚本
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh
# 安裝和更新 V2Ray
bash install-release.sh
# 安裝最新的 geoip.dat 和 geosite.dat
bash install-dat-release.sh
# 或使用apt安装（你可以试试看看软件包列表里有没有）
# sudo apt install v2ray
```

*注3: 若是从旧版本升级，迁移配置文件等你需要参考 [Migrate from the old script](https://github.com/v2fly/fhs-install-v2ray/wiki/Migrate-from-the-old-script-to-this)*

### 配置V

仅需要将 /usr/local/etc/v2ray/config.json 更改为以下内容：

``` json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": "your_port",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "your_id",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "your_path"
      }
    }
  },
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "block"
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "outboundTag": "block",
        "protocol": ["bittorrent"]
      }
    ]
  }
}
```

补充说明：

* your_port，请填写你的端口号，后续会用caddy将流量分发到此端口号上，比如3333选个你觉得舒服的，但不能填写被占用的端口（1000之前的数字）
* your_id，这是一个UUID，安装完毕后会自动生成，你也可以用[UUID Generator](https://www.uuidgenerator.net/)生成一个新的
* your_path，填写一个相对路由地址（e.g. /proxy, /ws, etc），caddy会捕捉这个路由地址的流量，转发到V中

### 开机自启动

``` bash
sudo systemctl enable v2ray
sudo systemctl restart v2ray
# 查看运行情况
sudo systemctl status v2ray -l
```

## Caddy

### 修改Caddyfile

接着就是要修改Caddyfile（/etc/caddy/Caddyfile），侦听your_path，转发到your_port中。

以下是修改完成后的Caddyfile

```
atomlab.org
tls xxxxx@mail.com
encode gzip # 相比v1发生了变化
log # 相比v1不再需要指定日志位置
root * /var/www/atomlab/public # 相比v1你需要增加一个*
file_server # 你需要指定以上目录，将caddy作为一个静态伺服器

# v2中你不再需要指定协议，caddy会自动升级成websocket
reverse_proxy your_path localhost:your_port 
```

### 重启Caddy服务

``` bash
sudo systemctl restart caddy
# 查看运行状态
sudo systemctl status caddy -l
```

## 写在最后

至此，服务端已经配置完毕，客户端的配置不再赘述了，如果是GUI的话挨个选项填写即可。关于客户端的选择，官方其实给出了许多[神一样的工具](https://www.v2ray.com/awesome/tools.html)，但可能被墙了你打不开，这里根据平台推荐几个我在用的。

* macOS: v2rayu，使用 homebrew cask 安装即可
* ubuntu/arch/other-linux: qv2ray, 使用对应的包管理器即可（ubuntu要用snap）
* ios/ipad: quantumult，需要美区ID才能购买下载，网上有如何注册美区ID的教程（但估计有些不好使了），在amazon上购买itunes充值卡即可购买
* android/windows: 我不用所以我不太清楚，你可以戳一下官方给出的工具们看看有哪些

*注：客户端的端口号要使用443，同时要启用tls并且选择websocket传输（重要），填写path，加密选择auto或者none即可（选择none，是因为我们走的是websocket+tls，数据已经被加密过了，不需要客户端再进行额外加密）。*

---
---
---

> ## 归档：Caddy v1 配置文件
> 
> 以下是修改完成后的Caddyfile（Caddy v1）
> 
> ```
> your_domain
> gzip
> tls your_email
> log /var/log/caddy/access.log
> root /var/www/atomlab/public
> proxy your_path localhost:your_port {
>     websocket
>     header_upstream -Origin
> }
> ```
