---
title: "博客系统构建（二）：V2ray"
date: 2018-07-23T15:56:43+08:00
draft: false
---

## 简介

### 这篇文章能带给你什么

* Project V + websocket + tls
* 利用caddy开放某个路由，通过websocket将流量转发到V2ray中
* cloudflare提供的免费cdn加速，隐藏真实ip地址

### 在开始之前，请确保

* 已经读过、或大致浏览过[博客系统构建（一）](/post/ladder/build-blog/)
* 有基本的Linux系统知识
* 能够使用vim/nano/emacs编辑文件
* 更新软件包和列表，到最新版本
* 文章中的命令均略去了sudo，若出现命令执行权限不够，你懂的

## Project V

### 什么是V？

[Project V](https://www.v2ray.com/)是一款用Go语言编写的新兴的网络分发、代理、混淆的自由软件。相比较老前辈SS，其配置更加丰富，能够利用它实现的功能会更加多，而对websocket/http2协议的支持，让它的穿透和抗检测能力大幅提高。

### 安装V

``` bash
# 首先安装依赖库
apt install unzip daemon
# 利用官方脚本安装
bash <(curl -L -s https://install.direct/go.sh)
```

### 配置V

仅需要更改/etc/v2ray/config.json中，inbound对应的值

``` json
"inbound": {
    "port": your_port,
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
}
```

补充说明：
* your_port，请填写你的端口号，后续会用caddy将流量分发到此端口号上
* your_id，这是一个UUID，安装完毕后会自动生成，你也可以用[UUID Generator](https://www.uuidgenerator.net/)生成一个新的
* your_path，填写一个相对路由地址（e.g. /proxy, /ws, etc），caddy会捕捉这个路由地址的流量，转发到V中

### 开机自启动

``` bash
systemctl enable v2ray
systemctl restart v2ray
# 查看运行情况
systemctl status v2ray -l
```

## Caddy

### 修改Caddyfile

接着就是要修改Caddyfile（/etc/caddy/Caddyfile），侦听your_path，转发到your_port中。

以下是修改完成后的Caddyfile

```
your_domain
gzip
tls your_email
log /var/log/caddy/access.log
root /var/www/atomlab/public
proxy your_path localhost:your_port {
    websocket
    header_upstream -Origin
}
```

### 重启Caddy服务

``` bash
systemctl restart caddy
# 查看运行状态
systemctl status caddy -l
```

## 写在最后

至此，服务端已经配置完毕，客户端的配置不再赘述了，挨个选项填写即可。

注：客户端的端口号要使用443，同时要启用tls，填写path，加密选择auto即可。

