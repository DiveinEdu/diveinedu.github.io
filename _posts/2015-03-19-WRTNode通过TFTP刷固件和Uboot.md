---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode通过TFTP刷固件和Uboot



配套前提：
设备上已经刷写了DiveinEdu为WRTNode提供的Uboot：


 ##A.  Ubuntu安装TFTP Server
Linux平台的TFTP Server有很多种，我们这选择`tftpd-hpa`这个软件。
####Step 1：Instal TFTP Server
我们可以打开终端或者打开Ubuntu软件中心都可以安装软件，我选择打开终端，
输入如下命令`sudo apt-get install tftpd-hpa`来安装。如果你选择Ubuntu软件中心，可以搜索`tftpd-hpa`来安装它；
####Step 2: Configure TFTP Server
`tftpd-hpa`的配置文件路径在这：`/etc/defualt/tftpd-hpa`,用管理员权限打开并编辑此文件，使得该配置文件内容如下(我用户名是openwrt)：

```ini
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/openwrt/tftproot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure --create"
```

该配置是让tftp服务器的文档根目录设置为我当前用户的主目录下的tftproot子目录，以及开启监听系统所有的网络接口。

####Step 3: Restart TFTP Server
创建tftp server的文档根目录：

```bash
mkdir -p $HOME/tftproot
```

运行下面的命令进行重启服务：

```bash
sudo service tftpd-hpa restart
```


##B. 进行TFTP刷写操作
在执行刷写操作之前，我们需要一些网络配置和文件准备操作：

- 1.设置我们的PC上Ubuntu系统的有线网卡IP地址为`192.168.1.2`
- 2.把WRTNode的固件文件拷贝到前面定义好的tftp文档根目录下，
命名为`wrtnode-firmware.bin`,此文件名只是举例。
- 3.把WRTNode的Uboot文件拷贝到同样的那个目录下，
命名为`wrtnode-uboot.bin`,此文件名只是举例。

准备工作做好之后就开始动手接好网线以及USB2TTL的串口线，进行后续的任务。

####Step 1：TFTP 刷固件
- 1 打开终端，运行`minicom`，设置好串口参数`1152008N1`，关闭硬件流控
- 2 给WRTNode通电开机，马上在`minicom`上输入2，进入TFTP刷写固件程序
- 3 输入DeviceIP为`192.168.1.1`
- 4 输入ServerIP为`192.168.1.2`
- 5 输入文件名为固件的文件名：`wrtnode-firmware.bin`
- 6 回车等待固件刷写完毕。


####Step 2：TFTP 刷Uboot
- 1 打开终端，运行minicom，设置好串口参数`1152008N1`，关闭硬件流控
- 2 给WRTNode通电开机，马上在minicom上输入9，进入TFTP刷写固件程序
- 3 输入DeviceIP为`192.168.1.1`
- 4 输入ServerIP为`192.168.1.2`
- 5 输入文件名为固件的文件名：`wrtnode-uboot.bin`
- 6 回车等待uboot刷写完毕。