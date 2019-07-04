---
title: SSH免密码登陆到远程服务器
tags: ["code"]
date: 2018-12-25T09:47:22+08:00
lastmod: 2018-12-27T10:36:22+08:00
draft: false
---

## 引言

如果你刚接触GNU/Linux，那么在连接远端服务器的时候一定被这些问题困扰着：

- 服务器的IP地址记不住，每次都要掏出小本子看一下，还要多看几眼看看输地对不对
- 密码过长难以输对，或者密码太过简单容易被碰撞攻击

不过幸好，前辈们在设计SSH的时候，就已经帮我们考虑到了这个问题。

## 文章内容

* 利用ssh生成rsa钥匙对，并将共钥传输给远端服务器
* 在bash配置文件里加入连接别名

## 生成公钥

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

## 将共钥传输给远端服务器

``` bash
# id@server 是你的用户名和服务器地址喔
ssh-copy-id id@server
# 按规定输入服务器密码后，成功的话能看到终端输出
# Number of key(s) added:        1
```
公钥储存在远端服务器这个文件里：.ssh/authorized_keys

## Bash设置别名

最后我们可以将连接的命令行再简化，利用Bash的别名（alias）提高键入速度。

``` bash
# 将别名写到.bashrc中，'ssh_alias'可以替换成任何你想要的名称
echo "ssh_alias='ssh id@server'" >> ~/.bashrc
# 应用更改
source ~/.bashrc
# 另外如果重启终端后失效，那么说明.bash_profile没有调用.bashrc，需要加入再执行以下命令
# echo 'if [ -f ~/.bashrc ]; then . ~/.bashrc; fi' >> ~/.bash_profile
```

## 写在最后

至此，就可以在终端里通过’ssh_alias’直接连接远端服务器啦！超级方便！
