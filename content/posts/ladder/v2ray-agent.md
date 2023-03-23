---
title: "博客搭建指南（二）：代理服务器"
tags: ["code"]
categories: ["博客搭建指南"]
date: 2018-07-23T15:56:43+08:00
draft: false
---

## 简介

### 这篇文章能带给你什么

* Xray + tcp xtls + websocket fallback
* 文章已经在第一版（2018）的基础上经过多次修订，现在配置更为简单（2020）
* 在2023年进行了更新，现在使用Xray的XTLS来加速访问
* 归档部分若无特殊需要请跳过，在未来我会移除它（暂时保留以供有些同学升级时作为参考）

### 在开始之前，请确保

* 已经读过、或大致浏览过[博客系统构建（一）](/post/ladder/build-blog/)
* 有基本的Linux系统知识
* 能够使用vim/nano/emacs编辑文件
* 更新软件包和列表，到最新版本
* 文章中的部分命令略去了sudo

## Project X

### 什么是Xray？

[Xray](https://github.com/XTLS/Xray-core)是从前辈V2ray中产生的一个分支项目，它的特点就是支持XTLS，能获得数倍于前辈的性能（这里面还有一些历史故事，有兴趣的可以去看看）

*注1：本文的内容基本没有大幅度改动，另外据说现在SS被查的很严（2020）*  
*注2：安装和更新自 Aug 2020 起，有略微变化，你可以在这里找到更多的信息 [fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray)*
*注3：现已经更新使用Xray，快速安装脚本详见[xray-install](https://github.com/XTLS/Xray-install)*

### 安装Xray

``` bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install-geodata
```

### 配置Xray

仅需要将 /usr/local/etc/xray/config.json 更改为以下内容：

```json
{
  "log": {
    "access": "/var/log/xray/access.log",
    "error": "/var/log/xray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "your_id",
            "flow": "xtls-rprx-direct",
            "level": 0,
            "email": "your_email"
          }
        ],
        "decryption": "none",
        "fallbacks": [
          {
            "dest": 1314, // 默认回落到 Xray 的 Trojan 协议
            "xver": 1
          },
          {
            "path": "/free", // 必须换成自定义的 PATH
            "dest": 2234,
            "xver": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "xtls",
        "xtlsSettings": {
          "alpn": [
            "http/1.1"
          ],
          "certificates": [
            {
              "certificateFile": "/usr/local/etc/xray/cert.crt",
              "keyFile": "/usr/local/etc/xray/cert.key"
            }
          ]
        }
      }
    },
    {
      "port": 1314,
      "listen": "127.0.0.1",
      "protocol": "trojan",
      "settings": {
        "clients": [
          {
            "password": "your_password",
            "level": 0,
            "email": "your_email"
          }
        ],
        "fallbacks": [
          {
            "dest": 8080
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
          "acceptProxyProtocol": true
        }
      }
    },
    {
      "port": 2234,
      "listen": "127.0.0.1",
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "your_id",
            "level": 0,
            "email": "your_email"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
          "acceptProxyProtocol": true, // 提醒：若你用 Nginx/Caddy 等反代 WS，需要删掉这行
          "path": "your_path" // 必须换成自定义的 PATH，需要和分流的一致
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

补充说明：

* your_id，这是一个UUID，安装完毕后会自动生成，你也可以用[UUID Generator](https://www.uuidgenerator.net/)生成一个新的
* your_path，填写一个相对路由地址（e.g. /proxy, /ws, etc），回落到websocket时使用

### 利用acme.sh申请证书

由于所有的流量目前现在都由Xray分发，这时候caddy的自动证书就变得不好用了，所以我们要手动配置好证书申请。如果遇到任何困难请参考这篇文章[XTLS证书管理篇](https://xtls.github.io/document/level-0/ch06-certificates.html#_6-1-申请-tls-证书)。注意：安装acme和使用时确保已经切换到root账户。

```bash
sudo apt update
sudo apt install socat
# make sure you are root
sudo su # enter root password
wget -O -  https://get.acme.sh | sh
. .bashrc
acme.sh --upgrade --auto-upgrade
```

进行正式申请前的测试，以免触发Let's Encrypt 的频率上限，需要先暂停caddy的服务以免占用80端口。

```bash
sudo systemctl stop caddy
acme.sh --issue --server letsencrypt --test -d your_domain --keylength ec-256 --standalone
```

如果没有出错，那么开始申请证书。 

```bash
acme.sh --set-default-ca --server letsencrypt
acme.sh --issue -d your_domain --keylength ec-256 --standalone
```

最后开始颁发证书到Xray中配置好的路径中，重新启动caddy。

```bash
acme.sh --installcert -d your_cert --cert-file /usr/local/etc/xray/cert.crt --key-file /usr/local/etc/xray/cert.key --fullchain-file /usr/local/etc/xray/fullchain.crt --ecc
sudo systemctl start caddy
```

### 开机自启动

``` bash
sudo systemctl enable xray
sudo systemctl restart xray
# 查看运行情况
sudo systemctl status xray -l
```

## Caddy

### 修改Caddyfile

接着就是要修改Caddyfile（/etc/caddy/Caddyfile），将http重定向到https的443端口（即转发给Xray）那么无论是http还是https请求最终都会进入Xray进行分发。以下是修改完成后的Caddyfile

```
}
  http_port 80
  https_port 8443
  auto_https disable_redirects
}

http://atomlab.org {
  log
  redir https://atomlab.org:443 permanent
}

http://atomlab.org:8080 {
  encode gzip
  root * /var/www/atomlab/public
  file_server
}
```

### 重启Caddy服务

``` bash
sudo systemctl restart caddy
# 查看运行状态
sudo systemctl status caddy -l
```

## 写在最后

至此，服务端已经配置完毕，客户端的配置不再赘述了，如果是GUI的话挨个选项填写即可。关于客户端的选择，前辈V2ray官方其实给出了许多[神一样的工具](https://www.v2ray.com/awesome/tools.html)，但可能被墙了你打不开，这里根据平台推荐几个我在用的。

* macOS: v2rayxs，使用 homebrew cask 安装即可
* ubuntu/arch/other-linux: qv2ray, 使用对应的包管理器即可（ubuntu要用snap）
* ios/ipad: quantumult，需要美区ID才能购买下载，网上有如何注册美区ID的教程（但估计有些不好使了），在amazon上购买itunes充值卡即可购买
* android/windows: 我不用所以我不太清楚，你可以戳一下官方给出的工具们看看有哪些

*注：客户端的端口号要使用443，最佳的配置应该是使用xray-core，使用VLESS协议，XTLS over TCP。*

*注：若使用websocket，同时要启用tls并且选择websocket传输（重要），填写path，加密选择auto或者none即可（选择none，是因为我们走的是websocket+tls，数据已经被加密过了，不需要客户端再进行额外加密）。*

---
---
---

> ## 归档：V2ray 配置文件
> 
> 仅需要将 /usr/local/etc/v2ray/config.json 更改为以下内容：
> 
> ``` json
> {
>   "log": {
>     "access": "/var/log/v2ray/access.log",
>     "error": "/var/log/v2ray/error.log",
>     "loglevel": "warning"
>   },
>   "inbound": {
>     "port": "your_port",
>     "protocol": "vmess",
>     "settings": {
>       "clients": [
>         {
>           "id": "your_id",
>           "level": 1,
>           "alterId": 64
>         }
>       ]
>     },
>     "streamSettings": {
>       "network": "ws",
>       "wsSettings": {
>         "path": "your_path"
>       }
>     }
>   },
>   "outbounds": [
>     {
>       "protocol": "freedom",
>       "settings": {}
>     },
>     {
>       "protocol": "blackhole",
>       "settings": {},
>       "tag": "block"
>     }
>   ],
>   "routing": {
>     "domainStrategy": "AsIs",
>     "rules": [
>       {
>         "type": "field",
>         "outboundTag": "block",
>         "protocol": ["bittorrent"]
>       }
>     ]
>   }
> }
> ```
> 以下是修改完成后的Caddyfile
> 
> ```
> atomlab.org
> tls xxxxx@mail.com
> encode gzip # 相比v1发生了变化
> log # 相比v1不再需要指定日志位置
> root * /var/www/atomlab/public # 相比v1你需要增加一个*
> file_server # 你需要指定以上目录，将caddy作为一个静态伺服器
> 
> # v2中你不再需要指定协议，caddy会自动升级成websocket
> reverse_proxy your_path localhost:your_port 
> ```

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
