---
layout: post
category : 原创
tagline: "WebRTC视频聊天"
tags : [WebRTC, 视频聊天, 开源框架, iOS, XMPP]
---
#第五章 WebRTC的iOS框架编译

##1.WebRTC的iOS框架的选择
目前两个比较活跃的开源WebRTC实现.

* Google WebRTC:

项目地址是: https://code.google.com/p/webrtc/

* Ericsson Research OpenWebRTC:

项目地址是: https://github.com/EricssonResearch/openwebrtc

我们[戴维营教育](http://www.diveinedu.cn)为了给学生实战项目中运用WebRTC视频通话技术，选择Google的WebRTC项目来构建iOS App的开发框架,因为目前Chrome浏览器和FireFox浏览器的WebRTC支持都是采用该项目.那么问题就来了,既然浏览器里都支持了WebRTC,那我们再去移植编译它到iOS平台干嘛呢,直接用webview 不行? 对,还不行! Apple在这方面已经严重拖后腿了.不过他有他牛逼的Facetime技术,可以随时随地的视频通话,但是他不开源,所以我们只能垂涎了. 故还是老老实实的移植WebRTC吧.非常幸运的是,Google 的Chromium项目开发者已经实现了其WebRTC的Objective-C的一套API了.


不过,丑话还是说在前头好,要从零开始集成WebRTC到我们的App中中, 简直就是噩梦;因为WebRTC项目和Chromium项目有一定的关联依赖关系,而且这些项目都是跨平台的大项目,采用了Google自己的一套编译系统,相对我们日常的IDE来说要复杂的多.如果我们需要得到一个WebRTC的库或者框架,我们就需要忘记Xcode IDE和Interface Builder这些高科技,我们要切换到终端环境下用命令行下的黑科技来征服这一切.


##2.开始WebRTC源码下载
前提条件:

* 我现在用的Macbook,8G内存,运行OS X 10.9.5.
* 安装最新的git和subversions并确保其可正常工作.
* Xcode 6.1.1 和 Command Line Tools.
* 中国大陆用户额外要求,快速的VPN,或者快速的shadowsocks服务.(翻墙和给git和svn以及curl设置代理等).

###2.1 创建一个编译目录
我们创建一个目录专门来存放项目编译工具和项目代码仓库等.确保该目录所在磁盘可用空间至少有8~10G.打开系统的终端工具进入到Shell:

```bash
wuqiong:~ apple$mkdir -p $HOME/opensource/webrtc_build/
```

###2.2 下载Chromium的depot工具
在执行下面命令之前,请确保你已经连上快速VPN已经翻墙了,或者你已经给git单独配置了有效的socks翻墙代理,如果你这些都不是问题,就当我没说.

```bash
wuqiong:~ apple$cd $HOME/opensource/webrtc_build/
wuqiong:webrtc_build apple$git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

这是一套Google用来编译Chromium或者WebRTC的构建工具,在我们后续的编译过程中也将使用它.为了命令行使用方便,我们把这些工具的路径加入到系统环境变量PATH中去:

```bash
wuqiong:webrtc_build apple$echo "export PATH=$PWD/depot_tools:$PATH" > $HOME/.bash_profile
```

然后需要关闭当前终端重新开启一个来上面设置的环境变量生效.或者在现在终端执行入门命令在当前终端里加载生效:

```bash
wuqiong:webrtc_build apple$source $HOME/.bash_profile
```

###2.3 下载WebRTC的源码
在我们的编译工作目录webrtc_build下创建一个webtrtc子目录来存放代码,请执行下面命令:

```bash
wuqiong:webrtc_build apple$ mkdir webrtc
wuqiong:webrtc_build apple$ cd webrtc 
```

在上面的检查工作没错之后,我们就需要开始把WebRTC项目的代码仓库下载一份到本地来.由于其仓库之大,大约一共需要下载6G+的东西.所以这一步非常需要有耐心.而且需要有稳定无障碍的互联网. 执行如下命令然后吧:

```bash
wuqiong:webrtc apple$ gclient config --name src http://webrtc.googlecode.com/svn/trunk
wuqiong:webrtc apple$ echo "target_os = ['ios']" >> .gclient
wuqiong:webrtc apple$ gclient sync --force
```

翻墙快的去喝咖啡,慢的去约妹子吧.办完事情之后回来如果上面的命令都一切顺利,我们就可以往下走去开始编译了.
(为了方便大家,我已经把webrtc_build目录打包备份,这样大家可以省去大量的代码下载时间.打包文件有5G,正在寻找网盘存放,随后公布.)

###2.4 编译WebRTC.framework
到了这一步,源码应该已经下载好了.这些源码可以编译为好几个平台,OS X, Linux, Windows, Android, iOS等.这里我们只需要编译iOS平台的WebRTC,并制作成一个iOS的开发框架.这里我们不能用Xcode工具,因为这些项目压根就不支持XCode.我们需要在终端命令行环境下去搞定这一切!

首先,为了我们装逼玩黑武器,我们需要在webrtc的项目代码目录下创建一个脚本, 这个脚本就是我为了简化命令的复杂度和提高使用的方便性专门编写的一个一键框架编译脚本,这个脚本就是今天的核心黑科技了.先创建一个空文件,然后赋予执行权限:

```bash
wuqiong:webrtc apple$ touch build_webrtc.sh
wuqiong:webrtc apple$ chmod +x build_webrtc.sh
```

然后用编辑器打开编辑刚刚创建的脚本文件,把如下脚本粘贴进去之后保存并关闭:

```bash
#!/bin/bash
# Script to build WebRTC.framework for iOS
# Copyright (C) 2015 戴维营教育  - All Rights Reserved
# Last revised 28/1/2015
#

function build_iossim_ia32() {
	echo "*** building WebRTC for the ia32 iOS simulator";
	export GYP_GENERATORS="ninja";
	export GYP_DEFINES="build_with_libjingle=1 build_with_chromium=0 libjingle_objc=1 OS=ios target_arch=ia32";
	export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_ia32";
	export GYP_CROSSCOMPILE=1;
	pushd src;
	gclient runhooks;
	ninja -C out_ios_ia32/Release-iphonesimulator iossim AppRTCDemo;

	echo "*** creating iOS ia32 libraries";
	pushd out_ios_ia32/Release-iphonesimulator/;
	rm -f  libapprtc_signaling.a;
	popd;
	mkdir -p out_ios_ia32/libs;
	libtool -static -o out_ios_ia32/libs/libWebRTC-ia32.a out_ios_ia32/Release-iphonesimulator/lib*.a;
	strip -S -x -o out_ios_ia32/libs/libWebRTC.a -r out_ios_ia32/libs/libWebRTC-ia32.a;
	rm -f out_ios_ia32/libs/libWebRTC-ia32.a;
	echo "*** result: $PWD/out_ios_ia32/libs/libWebRTC.a";

	popd;
}

function build_iossim_x86_64() {
	echo "*** building WebRTC for the x86_64 iOS simulator";
	export GYP_GENERATORS="ninja";
	export GYP_DEFINES="build_with_libjingle=1 build_with_chromium=0 libjingle_objc=1 OS=ios target_arch=x64 target_subarch=arm64";
	export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_x86_64";
	export GYP_CROSSCOMPILE=1;
	pushd src;
	gclient runhooks;
	ninja -C out_ios_x86_64/Release-iphonesimulator iossim AppRTCDemo;

	echo "*** creating iOS x86_64 libraries";
	pushd out_ios_x86_64/Release-iphonesimulator/;
	rm -f  libapprtc_signaling.a;
	popd;
	mkdir -p out_ios_x86_64/libs;
	libtool -static -o out_ios_x86_64/libs/libWebRTC-x86_64.a out_ios_x86_64/Release-iphonesimulator/lib*.a;
	strip -S -x -o out_ios_x86_64/libs/libWebRTC.a -r out_ios_x86_64/libs/libWebRTC-x86_64.a;
	echo "*** result: $PWD/out_ios_x86_64/libs/libWebRTC.a";

	popd;
}

function build_iosdevice_armv7() {
	echo "*** building WebRTC for armv7 iOS devices";
	export GYP_GENERATORS="ninja";
	export GYP_DEFINES="build_with_libjingle=1 build_with_chromium=0 libjingle_objc=1 OS=ios target_arch=armv7";
	export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_armv7";
	export GYP_CROSSCOMPILE=1;
	pushd src;
	gclient runhooks;
	ninja -C out_ios_armv7/Release-iphoneos AppRTCDemo;

	echo "*** creating iOS armv7 libraries";
	pushd out_ios_armv7/Release-iphoneos/;
	rm -f  libapprtc_signaling.a;
	popd;
	mkdir -p out_ios_armv7/libs;
	libtool -static -o out_ios_armv7/libs/libWebRTC-armv7.a out_ios_armv7/Release-iphoneos/lib*.a;
	strip -S -x -o out_ios_armv7/libs/libWebRTC.a -r out_ios_armv7/libs/libWebRTC-armv7.a;
	echo "*** result: $PWD/out_ios_armv7/libs/libWebRTC.a";

	popd;
}

function build_iosdevice_arm64() {
	echo "*** building WebRTC for arm64 iOS devices";
	export GYP_GENERATORS="ninja";
	export GYP_DEFINES="build_with_libjingle=1 build_with_chromium=0 libjingle_objc=1 OS=ios target_arch=arm64 target_subarch=arm64";
	export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_arm64";
	export GYP_CROSSCOMPILE=1;
	pushd src;
	gclient runhooks;
	ninja -C out_ios_arm64/Release-iphoneos AppRTCDemo;

	echo "*** creating iOS arm64 libraries";
	pushd out_ios_arm64/Release-iphoneos/;
	rm -f  libapprtc_signaling.a;
	popd;
	mkdir -p out_ios_arm64/libs;
	libtool -static -o out_ios_arm64/libs/libWebRTC-arm64.a out_ios_arm64/Release-iphoneos/lib*.a;
	strip -S -x -o out_ios_arm64/libs/libWebRTC.a -r out_ios_arm64/libs/libWebRTC-arm64.a;
	echo "*** result: $PWD/out_ios_arm64/libs/libWebRTC.a";

	popd;
}

function combine_libs() 
{
	echo "*** combining libraries";
	lipo  -create 	src/out_ios_ia32/libs/libWebRTC.a \
			src/out_ios_x86_64/libs/libWebRTC.a \
			src/out_ios_armv7/libs/libWebRTC.a \
			src/out_ios_arm64/libs/libWebRTC.a \
			-output libWebRTC.a;
	echo "The public headers are located in $PWD/src/talk/app/webrtc/objc/public/*.h";
}

function create_framework() {
	echo "*** creating WebRTC.framework";
	rm -rf WebRTC.framework;
	mkdir -p WebRTC.framework/Versions/A/Headers;
	cp ./src/talk/app/webrtc/objc/public/*.h WebRTC.framework/Versions/A/Headers;
	cp libWebRTC.a WebRTC.framework/Versions/A/WebRTC;

	pushd WebRTC.framework/Versions;
	ln -sfh A Current;
	popd;
	pushd WebRTC.framework;
	ln -sfh Versions/Current/Headers Headers;
	ln -sfh Versions/Current/WebRTC WebRTC;
	popd;
}

function clean() 
{
    echo "*** cleaning";
    pushd src;
    rm -rf out_ios_arm64 out_ios_armv7 out_ios_ia32 out_ios_x86_64;
    popd;
    echo "*** all cleaned";
}

function update()
{
    gclient sync --force
    pushd src
    svn info | grep Revision > ../svn_rev.txt
    popd
}

function build_all() {
	build_iossim_ia32 && build_iossim_x86_64 && \
	build_iosdevice_armv7 && build_iosdevice_arm64 && \
	combine_libs && create_framework;
}

function run_simulator_ia32() {
	echo "*** running webrtc appdemo on ia32 iOS simulator";
	src/out_ios_ia32/Release-iphonesimulator/iossim src/out_ios_ia32/Release-iphonesimulator/AppRTCDemo.app;
}

function run_simulator_x86_64() {
	echo "*** running webrtc appdemo on x86_64 iOS simulator";
	src/out_ios_x86_64/Release-iphonesimulator/iossim -d 'iPhone 6' -s '8.1'  src/out_ios_x86_64/Release-iphonesimulator/AppRTCDemo.app;
}

function run_on_device_armv7() {
	echo "*** launching on armv7 iOS device";
	ideviceinstaller -i src/out_ios_armv7/Release-iphoneos/AppRTCDemo.app;
	echo "*** launch complete";
}

function run_on_device_arm64() {
	echo "*** launching on arm64 iOS device";
	ideviceinstaller -i src/out_ios_arm64/Release-iphoneos/AppRTCDemo.app;
	echo "*** launch complete";
}

#运行命令行参数中第一个参数所指定的Shell函数
$@

```

这个编译脚本除了可以编译WebRTC项目自带的AppRTCDemo应用外,还可以编译出WebRTC.framework.

执行如下命令来编译我们所需要的全部:

```bash
wuqiong:webrtc apple$ ./build_webrtc.sh build_all
```
等上面命令完成之后,我们所需要的WebRTC框架就在当前目录下了.可以用`ls`命令查看之:

```bash
wuqiong:webrtc apple$ ls 
WebRTC.framework build_webrtc.sh  libWebRTC.a      src
wuqiong:webrtc apple$ 
```

第一个`WebRTC.framework`就是我们需要的框架了! 到此,我们的编译任务就完成了! 不是吧..就这么简单?不是说起来超级麻烦吗?呵呵,装逼结束. 繁琐的部分已经封装到了shell脚本里头去了.如果有兴趣可以去研究一下这个脚本.

###2.5 WebRTC.framework的依赖.
如果项目使用了该框架,那么编译的时候需要在项目的Build Phases中添加如下库和框架:

* libstdc++.6.dylib
* libsqlite3.dylib
* libc++.dylib
* libicucore.dylib
* Security.framework
* CFNetwork.framework
* GLKit.framework
* AudioToolbox.framework
* AVFoundation.framework
* CoreAudio.framework
* CoreMedia.framework
* CoreVideo.framework
* CoreGraphics.framework
* OpenGLES.framework
* QuartzCore.framework

###重要提示
目前Google官方代码中在ARMv7平台有VP8视频编码的stackoverflow问题，会直接导致程序崩溃，如需了解详情并获取补丁，请联系[戴维营教育](http://www.diveinedu.cn)。

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

