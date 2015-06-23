---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode_OpenWRT通过串口线Kermit协议刷固件

> 
由于各种原因我们不得不采用这个方式去救砖
设备上运行的固件出现问题：
- 我们在调试固件的时候网络功能启动失败，WiFi开启不了。
- 我们在调试固件的时候内核都Panic了。系统都起不来。

##方式:
通过TTL串口线（USB2TTL模块）链接电脑，通过minicom和ckermit这2给利器来帮我们完成任务

minicom的波特率要和板子uboot的波特率配置一致。
kermit的配置波特率一样都要一致，三者合一。


