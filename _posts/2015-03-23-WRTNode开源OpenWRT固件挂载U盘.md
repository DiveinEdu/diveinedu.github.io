---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, OpenWRT, WRTNode, 路由器]
---
#WRTNode平台OpenWRT-14-07固件挂载U盘



##OpenWRT系统环境介绍
由于OpenWRT官方稳定版BB固件默认集成的软件包以及内核驱动模块都非常少，
所以我们需要额外的去安装那些挂载U盘所必须的驱动模块和用户空间软件包。


##挂载U盘所需驱动分析
- 首先，U盘是一个接在USB总线上设备，故我们需要USB总线的驱动，在Linux内核代码中，USB总线的驱动集成到SCSI总线驱动的模块当中去啦。对应应该安装的驱动名是`kmod-scsi-generic`。USB控制器之类的驱动就是`kmod-usb-ohci,kmod-usb-uhci,kmod-usb2`
- 其次，U盘是一个USB总线上的磁盘存储设备，Linux内核驱动代码中，对应的驱动模块是`kmod-usb-storage`，如果是其他USB读卡器设备，驱动模块是`kmod-usb-storage-extras`。
- 然后，U盘上的磁盘分区有文件系统，根据上面的分区文件系统不同，需要不同的驱动模块，一般来说，我们用来在PC上和WRTNode来复制文件的U盘大都是微软的FAT32或NTFS文件系统，对应的Linux内核的驱动模块是`kmod-fs-vfat`以及`kmod-fs-ntfs`，如果要对U盘里NTFS分区进行写入，最好安装`NTFS-3G`驱动。
- 最后，我们的每种文件系统中，关于路径，文件名等数据的编码都是可以不同的，所以我们需要内核当中有我们U盘里文件系统中相应的编码模块，而在内核里的叫法叫做CodePage（内码表，代码页），我们现在Windows的FAT32文件系统的CodePage一般是代号437，后来微软在内码转向UTF-16之前新增定义了一系列的支持不同语言字符集的代码页，被称作"Windows（或ANSI）代码页"。代表性的是实现了ISO-8859-1的代码页1252。
其对应的2个内核模块是：`kmod-nls-cp437` `kmod-nls-iso8859-1`


##驱动模块和程序安装
根据前面的分析结果，我们用下面命令来安装这些必须的驱动和软件包：

```bash
opkg install kmod-scsi-generic kmod-usb-uhci kmod-usb-storage kmod-usb-storage-extras  kmod-fs-vfat kmod-fs-ntfs kmod-nls-cp437 kmod-nls-iso8859-1 ntfs-3g block-mount
```
安装完驱动之后，我们重启一下开发板或者路由器设备。


##U盘手动挂载操作
插入我们的U盘，同时会在串口终端上看到有USB设备变动的信息，会在系统的`/dev/`目录下生成磁盘设备的文件`sdaX`这样的。运行下面命令进行挂载，比如挂载sda4到系的/mnt命令下：

```bash
mount /dev/sda4  /mnt/
```
就这样，U盘的分区就成功挂载到了系统的`/mnt`目录下面来了。我们可以运行`ls /mnt`命令来列出U盘里的文件信息。


##U盘自动挂载操作
编辑这个文件`/etc/hotplug.d/block/10-mount`，使得其内容如下:

```bash
#!/bin/sh
# Copyright (C) 2009 OpenWrt.org  (C) 2010 OpenWrt.org.cn
/sbin/block hotplug

blkdev=`dirname $DEVPATH`
if [ `basename $blkdev` != "block" ]; then
device=`basename $DEVPATH`
case "$ACTION" in
add)
mkdir -p /mnt/$device
# vfat & ntfs-3g check
if  [ `which fdisk` ]; then
isntfs=`fdisk -l | grep $device | grep NTFS`
isvfat=`fdisk -l | grep $device | grep FAT`
isfuse=`lsmod | grep fuse`
isntfs3g=`which ntfs-3g`
else
isntfs=""
isvfat=""
fi

# mount with ntfs-3g if possible, else with default mount
if [ "$isntfs" -a "$isfuse" -a "$isntfs3g" ]; then
ntfs-3g -o nls=utf8 /dev/$device /mnt/$device
elif [ "$isvfat" ]; then
mount -t vfat -o iocharset=utf8,rw,sync,umask=0000,dmask=0000,fmask=0000 /dev/$device /mnt/$device
else
mount /dev/$device /mnt/$device
fi
if [ -f /dev/${device}/swapfile ]; then
mkswap /dev/${device}/swapfile
swapon /dev/${device}/swapfile
fi
;;
remove)
if [ -f /dev/${device}/swapfile ]; then
swapoff /dev/${device}/swapfile
fi
umount /dev/$device
;;
esac

fi

```


保存之后就可以进行U盘的插拔自动挂载啦。