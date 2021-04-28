---
title: "浙江大学有线网和树莓派路由器（二）"
tags: ["code"]
categories: ["树莓派"]
date: 2019-09-13T21:21:13+08:00
lastmod: 2019-09-13T21:21:13+08:00
draft: false
---

## 简介

在设置完无线热点以后，其实在宿舍里wifi信道干扰严重，主要是体现在极高的丢包率（over 50%）和延迟上。如果你的设备刚好支持通过以太网（有线）连接的话，可以和我一样购买usb->rj45的有线网卡给树莓派进行扩展。

**注意！这不是一个step-by-step的教程，而是作为系列一的补充启到抛砖引玉的作用，所以请不要copy-paste，也不要做全文件的替换！**

## 网卡选择

我为rpi3选择了rtl8152芯片的usb有线网卡，它能够免驱在rasbian（debian10）下工作，非常方便。这里多说一句，对于rpi4以下机型请选择百兆usb网卡，因为受限于usb2.0的通道速度，千兆网卡根本发挥不出它的性能，并且会存在兼容性问题。如果是rpi4以上机型，通道改为了usb3.0那么可以选择rtl8153芯片（网上说也免驱，我没有测试）。

在购买得到有线网卡以后，接上以后可以通过lsusb来查看rasbian是否成功识别了它，以下是rtl8152芯片的样例输出：

```
Bus 001 Device 006: ID 0bda:8152 Realtek Semiconductor Corp. RTL8152 Fast Ethernet Adapter
```

## 组网

### 方案设想

我购买了两个usb有线网卡，算上树莓派自身的无线和有线，那么总计有：eth0, eth1, eth2, wlan0，四个物理网卡。我组网设想的方案是将eth0作为wan口，然后搭建一个br0的网桥，将eth1, eth2, wlan0桥接在一起，为br0启用dhcp服务。最后，在完成浙大l2tp拨号后，通过iptables为ppp0设置nat（这一步其实在系列一里已经完成了）。

### 具体实现

首先更新软件包列表后安装依赖：bridge-util，我们要利用brctl建立网桥和管理br0中的interfaces（自己用apt安装即可）。

```bash
# 建立br0网桥
brctl addbr br0
```

有些不太一样的是，之前我们在/etc/dhcpcd.conf中是为wlan0设置了静态ip的，此时我们要将设备名改为br0，即：

```conf
interface br0
static ip_address=192.168.11.1/24
```

其次是在/etc/dnsmasq.conf中，设备名也要修改为br0，即：

```conf
interface=br0
dhcp-range=192.168.11.10,192.168.11.30,255.255.255.0,24h
```

最后，也要告诉hostapd使用br0网桥，即在/etc/hostapd/hostapd.conf中，要追加一行：

```conf
bridge=br0
```

OK，所有配置文件修改完毕，这里我建议大家先重启树莓派以让所有配置生效。重启完毕后，使用ifconfig查看应该会看到多出了一个br0的interface（同时你应该还要看到你的usb网卡，就是eth1）。那么先根据系列一提供的脚本拨号，让pi能够上网（这时会多出一个ppp0的interface），接着我们就要开始进行网络桥接：

```bash
brctl addif br0 eth1
# 若你没有第二张usb网卡，那么这行不要执行
brctl addif br0 eth2
```

至此，此时给你的设备接上网线，设备上设置dhcp自动获取ip地址，你就可以通过有线上网啦！

## 其他技术细节

### 开机自动组建网桥

每次手动都要组建网桥是非常麻烦的一件事，这部分我还在研究。难点主要是会牵扯网卡启动顺序，所以需要寻找一种合适的方案。此部分留空，后续我会更新。

## 参考资料

1. [Differences between /etc/dhcpcd.conf and /etc/network/interfaces?](https://raspberrypi.stackexchange.com/questions/39785/differences-between-etc-dhcpcd-conf-and-etc-network-interfaces)
2. [Linux Advanced Routing & Traffic Control HOWTO](https://lartc.org/howto/)
