---
title: "博客系统构建（三）：BBR+CDN"
date: 2018-07-23T15:58:03+08:00
draft: false
---

## 简介

### 这篇文章能带给你什么

* 升级Linux内核，开启拥塞控制算法（bbr）
* cloudflare提供的免费cdn加速，隐藏真实ip地址

### 在开始之前，请确保

* 已经读过、或大致浏览过[博客系统构建（一）](/post/ladder/build-blog/)和[博客系统构建（二）](/post/ladder/v2ray-agent/)
* 有基本的Linux系统知识
* 能够使用vim/nano/emacs编辑文件
* 更新软件包和列表，到最新版本
* 文章中的命令均略去了sudo，若出现命令执行权限不够，你懂的

## BBR

### 什么是BBR？

bbr是由google的某位工程师的茶余饭后作品，它能大幅度提高tcp的响应速度，加速服务器的响应和并发能力。值得一提的是，Linux的内核在4.9+版本中，已经支持开启tcp-bbr功能。

### 升级Linux内核

我的服务端系统采用的是Debian 8，故以此为例。

[Ubuntu/Debian官方内核列表](http://kernel.ubuntu.com/~kernel-ppa/mainline/)，选择内核版本时，首先是要linux-image-x.xx-generic开头的，其次是尽量选择已经签名的、比较新的版本，我这里选择了v4.16。

注意！在升级内核前请创建快照或者备份！！！

``` bash
# 下载v4.16内核
cd ~/Downloads
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.16/linux-image-4.16.0-041600-generic_4.16.0-041600.201804012230_amd64.deb
# 安装内核
dpkg -i linux-image-4.16.0-041600-generic_4.16.0-041600.201804012230_amd64.deb
# 若有必要，更新grub引导
update-grub
# 重启服务器
systemctl reboot
```

### 验证升级，开启bbr算法

``` bash
# 验证内核号，结果应为你选的版本号
uname -r
# 编辑/etc/sysctl.conf文件，加入以下两行
# net.core.default_qdisc=fq
# net.ipv4.tcp_congestion_control=bbr
# 或采用如下命令行
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
# 重载内核参数
sysctl -p
# 验证修改，输出结果应均带有bbr
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```

## CDN

cloudflare的免费CDN加速已经足够使用，若域名是在大陆备案的，则可以考虑使用七牛的融合CDN加速，效果更佳好。

cloudflare的配置十分简单，基本就是点击下一步下一步，到域名供应商那里修改以下name sever，最后等待几分钟后就完成了。其默认的配置不需要修改，不再赘述。

若采用其他家的CDN服务，保证tls版本，保证websocket服务打开即可。

## 写在最后

这个系列基本上就完结了，未来可能还会考虑实现博客文章推送后，自动化编译，利用workflow来实现最简单的后台管理等。