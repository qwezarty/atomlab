---
title: "SSH使用技巧-免密码/自动代理"
tags: ["code"]
date: 2020-10-16T21:19:44+08:00
draft: false
---

## Sec.0 引言

如果你刚接触GNU/Linux，那么在连接远端主机的时候一定被这些问题困扰着：

- 远端主机的IP地址记不住，每次都要掏出小本子看一下，还要多看几眼看看输地对不对
- 密码过长难以输对，或者密码太过简单容易被碰撞攻击
- 连接的网络环境复杂，每次要跳转多个堡垒机到达目标

不过幸好，前辈们在设计SSH的时候，就已经帮我们考虑到了这个问题。

## Sec.1 通过密钥来避免每次输入密码

### 什么是非对称加密算法

- 你生成了两把钥匙：“公共钥匙”和“私人钥匙”。公钥会被广泛地分发给其他任何想要和你通信的人；而私钥则由你自己保管。
- “钥匙对”使得通信过程中，不需要传递加密解密“规则”
	1. 你把公钥给了信任的人，他们将要发送的数据通过公钥加密
	2. 你拿到了这段数据，利用只储存在自己这里的私钥解密
	3. 中间数据即使被拦截，在没有唯一私钥的情况下，也没法解开数据包。
	4. 那么只要私钥不泄漏，通信就是安全的。
- 在数学原理上，阮一峰的博文[RSA算法原理（一）](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)做了很好的阐述，这里不再展开，有兴趣的同学可以了解一下。

### 生成钥匙对

``` bash
# 生成key，但首先你得安装ssh
ssh-keygen -t rsa -b 2048
# 一路按回车换行，终端输出大概是这样
# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/username/.ssh/id_rsa): 
# Enter passphrase (empty for no passphrase): 
# Enter same passphrase again: 
# Your identification has been saved in /home/username/.ssh/id_rsa.
# Your public key has been saved in /home/username/.ssh/id_rsa.pub.
```

### 将共钥传输给远端主机

``` bash
# id@server 是你的用户名和远端主机地址喔
ssh-copy-id id@server
# 按规定输入远端主机密码后，成功的话能看到终端输出
# Number of key(s) added:        1
```
公钥储存在远端主机这个文件里：.ssh/authorized_keys，现在当你重新再通过SSH连接到远端主机时，已经不再需要密码啦！

## Sec.2 如何优雅地透过堡垒机A连接主机B

OpenSSH 7.3 (later 2016) 版本以后自动支持了通过Proxy（远端主机A）来连接另一台远端主机B，这里说两种方法。

第一种是你临时使用的，一行命令行就可以搞定了：

```bash
# bastion是代理（主机A），remote是你真正要连的主机B
# 如果用默认端口22的话就可以忽略port不写
ssh -J user@<bastion:port> <user@remote:port>
```

第二种则是长期使用时，需要对 ~/.ssh/config 进行一些修改：

```
Host [alias-bastion]
  HostName [bastion-host-address]
  User [bastion-username]

Host [alias-remote]
  HostName [remote-host-address]
  User [remote-username]
  ProxyJump [alias-bastion]
```

DONOT COPY AND PASTE! 请根据你自己的信息替换掉所有"[...]"的内容（包括[]本身），如果你不知道怎么写，在下一个章节我会举一个例子。

正确地配置完成以后，直接就可以在终端里通过 "ssh alias-remote" 来连接到远端主机B了，SSH会自动帮你做隧道穿越。

## Sec.3 举个例子

我最近的需求是通过中国的高能所主机（bastion），连接到CERN的CMS主机（remote）。你首先需要保证这两个主机都能够独立连接上，附上高能所的帮助文档，包含如何申请账户：[高能所计算环境使用手册](http://afsapply.ihep.ac.cn/cchelp/zh/)

高能所的lxslc7节点们都是支持使用密钥来登陆的，所以我先把公钥给了bastion，能够避免一次密码输入。

```bash
ssh-copy-id ylye@lxslc7.ihep.ac.cn
```

在此之前，请先保证这两个主机你都能够独立连接上，我选择了第二种方案：写入到配置文件：下面是我的 ~/.ssh/config


```
Host ihep
  HostName lxslc7.ihep.ac.cn
  User ylye

Host cms
  HostName lxplus.cern.ch
  User yey
  ProxyJump ihep
```

配置完成以后，连接到CMS也变得很简单：

```bash
ssh cms
```

额外再提一句，SCP（Secure copy）本身是依赖SSH进行的，所以这样配置以后，文件传输的方法也变得很容易：

```bash
scp localfile cms:~/remote_folder
```

## 写在最后

其实所有常用的远端主机都可以通过这种方式来设置别名，快速地进行连接。而早在2017年之前，那个时候的方案则是用 bash alias 来实现的，其实比较麻烦。代理前/后的延迟分别是>450ms/150ms，肉眼可见的速度提升！（皮一下：喜欢我的文章也不需要关注我，也不用点赞）

HAVE FUN!

## Sec.4 参考资料

- [OpenSSH Config File Examples](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)
- [SSH to remote hosts though a proxy or bastion with ProxyJump](https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump)
- [Forward SSH through SSH tunnel](https://serverfault.com/questions/341190/forward-ssh-through-ssh-tunnel)
