---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode通过Uboot下USB刷固件



前提：
设备上已经刷写了DiveinEdu为WRTNode提供的Uboot：


##仅仅只是启动U盘里的固件:
用minicom通过TTL串口线连接至WRTNode按4进入Uboot命令行。
- 先熟悉几个Uboot的USB命令
 usb start
 其他由help usb查看具体命令

- FAT文件系统相关
 fatls
 fatinfo
 fatload

 fatload usb 0 0x80c00000 root_umage
 bootm 0x80c00000


##U盘刷写更新固件：
用minicom通过TTL串口线链接至WRTNode按5进入U盘自动刷机模式
（必须把固件命名为***`root_uImage`***放到U盘根目录下）
