---
title: "博客搭建指南（一）：静态内容"
tags: ["code"]
categories: ["博客搭建指南"]
date: 2018-07-23T13:32:57+08:00
draft: false
---

## 简介

### 这篇文章能带给你什么

* 使用Hugo生成静态博客
* 利用Caddy将博客通过HTTPS伺服起来
* 文章已经在第一版（2018）的基础上经过多次修订，现在配置更为简单（2020, CaddyV2）
* 在2023年进行了更新，现在使用XRay的XTLS来加速访问
* 归档部分若无特殊需要请跳过，在未来我会移除它（暂时保留以供有些同学升级时作为参考）

### 在开始之前，请确保

* 有基本的Linux系统知识
* 能够使用vim/nano/emacs编辑文件
* 拥有一个独立域名，一台Linux服务器（本文以Ubuntu20.04为例）
* 已经将域名解析到了Linux服务器上
* 更新软件包和列表，到最新版本
* 文章中的部分命令略去了sudo

## Hugo

### 什么是Hugo

[Hugo](https://gohugo.io/)是用Go语言编写的静态网站生成器（Static Site Generator），有不少静态模板，非常间接便于使用。

### 安装Hugo

在Ubuntu下安装Hugo就变得十分地便捷了（相比Debian而言），你只需要更新一下软件包安装即可。

```bash
sudo apt update
sudo apt install hugo
```

### 新建站点

``` bash
# 站点存放目录，与后面Caddy的配置联动
mkdir -p /var/www
cd /var/www
# 新建站点，本文以atomlab为例
hugo new site atomlab
```

### 安装Hugo主题

[Hugo的主题页](https://themes.gohugo.io/)，选择自己喜欢的一款主题，请先详细阅读一遍主题页的说明（非常重要），然后按照主题详情页的引导安装。本文以主题[even](https://themes.gohugo.io/hugo-theme-even/)为例。

``` bash
# 切换到新建好的站点中（你的站点名）
cd /var/www/atomlab
# 安装主题（不同主题安装方式不同）
git clone https://github.com/olOwOlo/hugo-theme-even themes/even
cp themes/even/exampleSite/config.toml ./config.toml
```

### 新建文章，测试主题

``` bash
# 假定工作目录在/var/www/atomlab
# 因为主题原因是post，一般而言是posts/hello.md
hugo new post/hello.md
# 生成静态网页，包括草稿，生成好的内容在public目录中
hugo -D
```

## Caddy

### 什么是Caddy

[Caddy](https://caddyserver.com/)是利用Go语言编写的较新的Web伺服器，比起老牌的Nginx，它的配置更为简单，并且能够自动在Let's Encrypt上更新HTTPS证书。另外，Caddy v2 更为完善，配置也更为方便和科学。

*注：本文安装的是 Caddy v2，作为参考我将 Caddy v1 归档处理。*

### 安装Caddy v2

根据官方的指引 [Download Caddy](https://caddyserver.com/docs/download)，只需要添加软件包地址再安装即可。

```bash
# 添加caddy软件包地址
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
# 更新软件包列表
sudo apt update
# 安装
sudo apt install caddy
```

*注：Caddy v2 已经不再需要配置很多文件权限和第三方自启动工具（没错，v2全部自带了，更加完善了）。*

### 编写Caddyfile配置文件（Caddy v2）

Caddyfile的位置并没有变，依然是 /etc/caddy/Caddyfile 

```
atomlab.org
tls xxxxx@mail.com
encode gzip # 相比v1发生了变化
log # 相比v1不再需要指定日志位置
root * /var/www/atomlab/public # 相比v1你需要增加一个*
file_server # 你需要指定以上目录，将caddy作为一个静态伺服器
```

以下是补充说明：

* 域名和tls的拥有者邮箱请改成自己的（会用这个邮箱申请证书）
* root目录要指向到站点生成下的public文件夹
* 若是你在短时间内过多次申请了证书（比如申请成功了证书，又多次摧毁了服务器重新建，那么你很可能会遇到免费的HTTPS证书申请不下来的情况，需要等待一段时间再试，详情请看 [Rate Limits](https://letsencrypt.org/docs/rate-limits/)）

## 写在最后

到这里，应该能够通过你的域名访问到博客了，请测试一下，不行的话主要看下域名解析和caddy的配置是否有错误，下一篇的内容是喜闻乐见的搭建XRay!

---
---
---

> ## 归档：Debian系统的安装流程
> 
> *以下内容是在Debian中安装Hugo的归档，作为参考，若无特殊需要请跳过。*
> 
> [Hugo的Release](https://github.com/gohugoio/hugo/releases)，请根据自己的需求选择不同版本。以下是在Linux服务端安装Hugo。
> 
> ``` bash
> # 首先下载Hugo
> mkdir ~/Downloads
> cd ~/Downloads
> wget https://github.com/gohugoio/hugo/releases/download/v0.45/hugo_0.45_Linux-64bit.tar.gz
> # 仅仅解压Hugo主程式
> tar -zxvf hugo_0.45_Linux-64bit.tar.gz hugo
> # 将程式丢到可运行目录中
> mv hugo /usr/local/bin
> # 测试安装
> hugo version
> ```
> 
> #### Debian安装Caddy v1，并配置
> 
> [Caddy的下载页](https://caddyserver.com/download)，根据自己的平台和要求，选择下载。本文是通过官方提供的Bash脚本安装。
> 
> ``` bash
> # 利用官方提供的脚本，安装caddy
> cd ~/Downloads
> curl https://getcaddy.com | bash -s personal
> # 测试安装
> caddy --version
> ```
> 
> 赋予caddy一些目录权限
> 
> ``` bash
> # 建立配置文件，更改文件所有权
> mkdir /etc/caddy
> touch /etc/caddy/Caddyfile
> chown -R root:www-data /etc/caddy
> # caddy自动获得的https证书存放位置
> mkdir /etc/ssl/caddy
> chown -R www-data:root /etc/ssl/caddy
> # 私钥禁止其他用户访问
> chmod 0770 /etc/ssl/caddy
> # 建立日志目录，给与写入权限
> mkdir /var/log/caddy
> touch /var/log/caddy/access.log
> chown -R www-data:root /var/log/caddy
> chmod 0666 /var/log/caddy/access.log
> ```
> 
> 开机自启动
> 
> ``` bash
> # 从官方的github仓库下载caddy服务
> curl -s https://raw.githubusercontent.com/mholt/caddy/master/dist/init/linux-systemd/caddy.service -o /etc/systemd/system/caddy.service
> # 刷新服务列表
> systemctl daemon-reload
> # 开机自启并启动caddy
> systemctl enable caddy
> systemctl start caddy
> # 查看运行详情，若出现错误，估计就是权限问题
> systemctl status caddy -l
> ```
> 
> 这里需要注意一下，若是Debian9，你需要做一些额外的工作才能让caddy顺利绑定443端口！
> 
> ```bash
> # 安装 libcap
> apt install -y libcap2-bin
> # set capability
> setcap CAP_NET_BIND_SERVICE=+eip $(which caddy)
> ```
> 
> #### 编写Caddyfile配置文件（Caddy v1）
> 
> ```
> atomlab.org
> gzip
> tls xxxxx@mail.com
> log /var/log/caddy/access.log
> root /var/www/atomlab/public
> ```
> 
> 以下是补充说明：
> 
> * 域名和tls的拥有者邮箱请改成自己的（会用这个邮箱申请证书）
> * root目录要指向到站点生成下的public文件夹

