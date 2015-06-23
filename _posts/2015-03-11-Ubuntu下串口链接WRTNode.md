---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#Ubuntu下串口链接WRTNode
####1. 下载MiniCom软件
sudo apt-get install minicom

####2. 设置当前用户的串口读写权限
sudo adduser $USER  dialout

####3. 设置Minicom的串口参数
 115200  8N1

####4. 通过串口控制台操作WRTNode

bla ... bla ...  bla...