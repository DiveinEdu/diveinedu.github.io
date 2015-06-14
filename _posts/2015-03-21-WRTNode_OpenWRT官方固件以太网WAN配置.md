---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode_OpenWRT官方固件以太网WAN配置



##下载官方固件并快速刷机:
- 官方开源稳定版本固件(Barrier Breaker 14.07版本)下载地址

```bash
cd $HOME/tftpdroot
wget http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620n/openwrt-ramips-mt7620n-wrtnode-squashfs-sysupgrade.bin  -O wrtnode-firmware.bin
```

- 用之前讲解过的tftp刷机方式快速刷机到官方开源固件


##编辑网络配置，使以太网口为WAN接口
OpenWRT官方的WRTNode固件中，WAN网络配置的默认情如下：

```bash
config interface 'wan'
        option ifname 'eth0.2'
        option force_link '1'
        option macaddr '64:51:7e:33:9b:7d'
        option proto 'dhcp'

config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '1 2 3 4 6t'

config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '0 6t'
```
在MT7620芯片中的以太网交换机可以对端口进行分组管理，默认的官方配置中，port 0配置为WAN口，而在WRTNode的那个标准底板上的唯一的那个以太网口却是port 3.所有默认的配置下，这个网口属于一个内网（LAN）接口， 现在我们需要把它修改为WAN接口，修改的方法如下：
把`option ports '1 2 3 4 6t'`中的`3`和`option ports '0 6t'`中的`0`进行位置交换就OK啦。

在串口终端上vim编辑该配置文件`/etc/config/network`后保存，然后重启网络就搞定。
如果需要给WAN口设置静态IP地址的话，需要修改`config intertace 'WAN'`下面的配置项目了,这个后面再讨论。


#本节之后的后续讲解都是在本节最初所刷机的OpenWRT官方的WRTNode固件(Barrier Breaker 14.07)的环境下讲解。