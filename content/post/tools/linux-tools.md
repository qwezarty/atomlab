---
title: "linux常用工具和软件"
tags: ["linux"]
date: 2019-07-04T14:54:57+08:00
lastmod: 2019-07-04T14:54:57+08:00
draft: false
---

## 引言

一直在Linux上工作，当更换新电脑或者发行版时，往往会忘记曾经在背后稳定默默工作着的“好家伙”们。我希望把这些好的开源项目工具都收录起来，甚至编写成一键安装脚本。我的系统是基于ArchLinux的，所以许多包会采用AUR的方式记录在此。我相信即便是其他发行版，你也能够找到相应的办法编译安装的。最后，这里也顺便推荐一下基于Arch的发行版[Manjaro with XFCE](https://manjaro.org/)，我非常喜欢它的高效和稳定。

## 系统工具

### v2ray

[Project V](https://www.v2ray.com/)，相当现代化的梯子，同时可以作为客户端和服务端，甚至是一个总的网络代理分发程序。服务端搭建详情可以参考博文[博客系统构建（二）：V2ray
](https://atomlab.org/post/ladder/v2ray-agent/)。

- Package: v2ray（Arch官方仓库带有此软件，可以直接使用pacman安装）
- Github: https://github.com/v2ray/v2ray-core

### motrix

[motrix](https://github.com/agalwood/Motrix)，很漂亮的下载工具，底层是调用了aria2的。支持magnet和torrent。但是因为是基于electron制作的，编译稍显繁琐（又是npm，我的天），值得一提的是作者还给国内IP加入了NPM镜像加速功能。

- AUR: https://aur.archlinux.org/motrix.git
- Github: https://github.com/agalwood/Motrix

### proxychains

[proxychains-ng](https://github.com/rofl0r/proxychains-ng)，能够使某一行命令全部走http（或者socks）代理。相当取巧好用的玩意儿，配合下面的privoxy食用更加舒适。

- Package: proxychains-ng
- Github: https://github.com/rofl0r/proxychains-ng

### privoxy

[privoxy](https://wiki.archlinux.org/index.php/Privoxy)，能将socks协议代理转换成http协议并监听。配合bash环境变量，这样做能让你在终端里挂起全局代理，十分推荐的东西。

- Package: privoxy

### virtual box

[virtual box](https://wiki.archlinux.org/index.php/VirtualBox)，当你需要运行一个windows虚拟机的时候（希望不会有这种时刻），你会需要用到它。它比vmware更加轻量稳定，记得若需要安装64位系统则需要在主板里打开处理器的虚拟化服务。

- Package: virtualbox

## 编辑器

### vim

vim大法好，大法好，好！多的不说了，我日常的编辑工具。哦对了，你最好安装gvim因为这样能提供系统级的剪切板，重要！

- Package: gvim

### vscode

Uh-huh，每次使用微软家的东西就让我很纠结。不喜欢使用vim的朋友，推荐给你们吧，号称宇宙第二的IDE（第一当然是VisualStudio，这玩意儿很贵并且只能运行在win上）。vscode开源免费，使用MIT许可。我只有在调试很复杂的程序的时候才会打开它。

- Package: code
- AUR: https://aur.archlinux.org/visual-studio-code-bin.git
- Github: https://github.com/microsoft/vscode

## 国外主流服务

### onedrive

[OneDrive Free Client](https://github.com/abraunegg/onedrive)，一个onedrive的命令行版。需要用你微软ID生成的KEY授权后使用，详见Github的项目主页。

- AUR: https://aur.archlinux.org/onedrive-abraunegg.git
- Github: https://github.com/abraunegg/onedrive

## 国内主流服务

### 网易云音乐

- AUR: https://aur.archlinux.org/netease-cloud-music.git

### 微信

[Electronic WeChat](https://github.com/geeeeeeeeek/electronic-wechat)，这是你能在Linux平台上找到的最漂亮的微信了，它是基于Electron制作。总工期超过1000天，可惜的是目前已经不再维护（截止Nov 12, 2018），可是我还没找到它的替代品，依然强烈推荐！

- AUR: https://aur.archlinux.org/electronic-wechat.git
- Github: https://github.com/kooritea/electronic-wechat

## 写在最后

想到了其他的会继续更新，以上。

