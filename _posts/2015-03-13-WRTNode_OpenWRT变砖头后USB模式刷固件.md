---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode/OpenWRT变砖头后USB模式刷固件


由于各种原因可能导致WRTNode
的系统启动不了之后的抢救办法

前提：Uboot刷用我们的Uboot，Uboot更改方法前面有演示

##Step1:
把准备好的没有问题的固件复制到U盘（FAT32文件系统）的根目录下，
重命名为root_uImage后，插入WRTNode底板的USB口


##Step2：
用串口连接WRTNode打开minicom控制台（Console），设置好参数115200  8N1


##Step3:
上电，minicom终端按 5

##Step4

坐等。。。

