---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#OpenWRT设置QOS限速和iPerf测速



##1.限速功能对应软件包安装

```bash
opkg install qos-scripts luci-app-qos
```
上面的命令会自动去安装它所依赖的iptables相关内核模块和TC模块等。
TC就是Linux内核内置的一个Traffic Control框架,可以实现流量限速,流量整形,策略应用(丢弃,NAT等)。


##2.设置全局限速

安装完成之后，编辑QOS配置文件`/etc/config/qos`，把下面配置打开

```bash
# QoS configuration for OpenWrt

# INTERFACES:
config interface wan
        option classgroup  "Default"
        option enabled      0
        option upload       128
        option download     1024
```

上面是最前面的配置，WAN的总出口限速设置，`enabled`改为`1`打开QOS， `upload`上行带宽设置，默认`128Kbps`，`download`选项是下行带宽，默认`1024Kbps`。那么默认就是总下行带宽限制为1M，上行带宽限制为128K。
如果我们要修改带宽，设置为总带宽是限制为对称的4M，那么我们应该修改`upload`为`4096`和`download`为`4096`。

修改完之后保存配置，然后重启`QOS`服务

```bash
/etc/init.d/qos restart
```


##3.利用iPerf工具进行带宽测速
iPerf是一个网络性能测试工具。iPerf可以测试TCP和UDP带宽质量。iPerf可以测量最大TCP带宽，具有多种参数和UDP特性。iPerf可以报告带宽，延迟抖动和数据包丢失。利用iPerf这一特性，可以用来测试一些网络设备如路由器，防火墙，交换机等的性能。
如果本机和服务器没有安装iPerf工具，Ubuntu/Debian系统用如下命令安装：

```bash
sudo apt-get install iperf
```

我们现在用它来测试我们的限速功能是否OK。
- 首先让本机通过WRTNode的WiFi上网，这样，我们本地的网络通过WRTNode，WRTNode通过唯一的那个底板上的网卡连到上级路由，上级路由和内网服务器链接。
- 然后登录到内网Linux服务器上运行iPerf服务端:
- 最后在本机新开一个终端，在本机终端运行iPerf客户端链接服务器进行测速，稍等片刻查看测速结果进行检验QOS限速功能。
- 可以修改限速带宽配置，多测试两次进行采样检验。



##4.这里仅仅是QOS限速的一个最简单使用，高级的限速用法以后讨论。