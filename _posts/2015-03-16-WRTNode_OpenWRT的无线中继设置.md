---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode_OpenWRT的无线中继设置

> 
由于各种原因我们不得不设置我们的路由器作为中继，
来扩大无线网络的覆盖范围

前提：
设备上运行的固件必须：
- 不是openwrt官方固件，因为官方估计默认没有集成web管理界面
- 不是WRTNode官方固件，因为官方固件不是用的开源驱动


##方式:
通过浏览器登录WRTNode或者OpenWRT的web管理后台的WiFi菜单进行设置

- 先删除所有的无线配置项目
- 再扫描设备周围存在的无线网络
- 然后选择要中继的目标网络，设置相应的sta模式配置信息
- 再新建一个AP模式的网络来为我们的手机提供接入点


