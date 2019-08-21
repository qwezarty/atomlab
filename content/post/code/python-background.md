---
title: "如何优雅地让Python程序在后台运行"
tags: ["code", "python"]
date: 2019-08-21T11:50:32+08:00
lastmod: 2019-08-21T11:50:32+08:00
author: "qwezarty"
draft: false
---

这篇短小的文章简单介绍了如何利用nohup将python脚本置于后台运行，即便你断开ssh连接也可以。另一方面也包含了如何利用I/O重定向保存日志文件、如何停止该后台程序等内容。

## nohup and &

nohup是linux的一个内置工具，能够使某条命令免疫挂起，并且将stdout保存到某个文件中。你还需要在末尾加上&符号，这是linux中的一种标准写法，意为：将这条命令置于后台运行。随后该进程可以简单地使用ps来看到。

```bash
# 后台运行，将stdout输出到nohup.out文件
nohup python scripts.py &
```

## output buffering

这是一个极容易遇到的坑！如果你细心观察，会发现如果python程序没有执行完毕（比如简单的http服务器开启监听），那么nohup.out文件中是一片空白的！WTF？怎么回事？这是python自己的标准输出机制，若想要关闭输出缓存则需要使用-u的flag，如下：

```bash
# 关闭输出缓存，实时记录到nohup.out中
nohup python -u scripts.py &
```

## I/O重定向

现在我们希望将stdout记录到某个指定的文件中，而不只是nohup.out，该如何处理？如果你没有I/O重定向的相关知识，我推荐你首先阅读[I/O Redirection](http://linuxcommand.org/lc3_lts0070.php)。

```bash
# 将标准输出放到name.log中
nohup python -u scripts.py > name.log 2>&1 &
```

这里我做进一步的解释，首先它将stdout重定向到了name.log中，其次又将stderr重定向到了stdout中，最后置于后台运行。这样无论是标准输出还是错误输出，都会进入到name.log文件中。

## 查看并关闭后台进程

OK，在成功创建了后台任务以后，我们可以简单地使用ps和kill来管理这些进程（多说几句，再推荐两个内置工具：pgrep和pkill，大家自己去看man page）。但是在此我们谈一些稍微复杂点的情况，如果你是将上述的nohup命令写在了某份bash脚本中，自动化执行的话（非交互式地运行），你通过简单的ps命令是根本找不到它的！

这个时候我们就要在执行nohup的时候，将它的输出（也就是该进程的pid）保存到文件中，最后在想要关闭该进程的时候读取它。

```bash
# 将标准输出放到name.log中，并且记录该进程的pid到run.pid中
nohup python -u scripts.py > name.log 2>&1 & echo $! > run.pid
```

这里再做一些补充：首先是echo，因为之前的命令已经置于后台运行了，所以从echo开始已经算作是一条新命令了，故不需要任何的前缀。其次是echo中的$!，它代表上一条命令的执行进程ID，所以我们将这个pid直接写入到run.pid即可。

加载run.pid并向该进程发送terminate信号也很容易，如下，不再详细说明。

```bash
# 若存在run.pid文件，则加载它并杀掉该进程
[[ -f run.pid ]] && kill $(cat run.pid)
```

## 结束语

linux中将程序置于后台运行，虽然看起来繁复，但是其实它设计的很合理，学会了以后也的确是非常高效。在知道原理和一些I/O重定向基础后，现在我们可以将任何一个可执行程序放在后台运行，并用ps就可以管理它们。最后，希望本文能够给你带来一些收获和思考，以上。

