---
layout: post
category: 原创
tagline: "Supporting tagline"
tags: [Mac,WebRTC,Shadowsocks,翻墙,代理,polipo,http_proxy,socks]
---



####1.polipo和shadowsocks简介
shadowsocks代理我想大家是耳熟能详了,在这里就不作过多的讲述.
不过有一点是需要指出的是,shadowsocks的1080代理端口的代理服务类型是传输层的socks代理.有些时候我们需要利用shadowsocks这个科学上网的隧道,同时需要HTTP代理的时候,我们就不得不想办法了.
我在这里就介绍另一款开源软件--`polipo`,一个小型快速的HTTP Web缓存代理服务器,还可以指定上级代理为socks类型.我们就利用这一点功能来实现我们的需求:
把shadowsocks的代理转换为http代理,让socks代理和http代理并行为我们提供科学上网服务.
这样尤其使得在unix-like系统中,很多命令行网络程序默认只能支持通过指定`http_proxy`环境变量来设置网络代理.这样利用`polipo`就把我们人人喜欢的shadowsocks服务科学上网功能扩展到http代理服务了.

####2. socks转http代理需求和动机
这个功能在今天我更新`WebRTC`最新代码的时候,执行`gclient sync`命令的时候中间会有一部分网络下载不是通过git来更新的,所以设置git的socks代理也无效,那一部分是执行`download_from_google_storage`下载更新的,他是python的boto代码,不支持一般的代理环境变量`http_proxy`设置方法.只能在`$HOME/.boto`文件中定义HTTP代理配置.

先运行shadowsocks服务的客户端,开启科学上网功能,自动和全局无所谓.



####3. 安装polipo


- Mac OS X平台安装

    请确保您系统安装了brew,然后用`brew`命令来安装polipo软件包:
    
    ```bash
    $ brew install polipo
    ```
    
    就这样一条命令就可以安装好.

- Ubuntu 平台安装

    Ubuntu Linux系统下安装方法如下:
    
    ```bash
    
    $ apt-get install polipo
    $ service polipo stop
    ```

    安装好之后,就可以运行`polipo`并设置上级socks代理为shadowsocks端口,就成功把socks转换为http代理.如shadowsocks的本地端口一般为`localhost:1080`:
    
    ```bash
    $ polipo socksParentProxy=localhost:1080
    Established listening socket on port 8123.
    ```
    
    
    
    
####4. 设置boto配置文件
用户目录下新建文件`$HOME/.boto`,内容如下命令所示:

```bash
$ cat $HOME/.boto 
[Boto]
proxy = 127.0.0.1
proxy_port = 8123
```

`127.0.0.1:8123`是polipo命令在本地运行的HTTP代理服务默认IP端口.



####5. 更新WebRTC代码

上面配置了boto的配置文件后,我们还需要导出一个环境变量,使该配置文件可以被识别生效:

```bash
$ export NO_AUTH_BOTO_CONFIG=$HOME/.boto
```
到这个时候,我们就可以继续执行`gclient sync`来更新WebRTC的所需`chromium`代码仓库了.

如果不导出这个`NO_AUTH_BOTO_CONFIG`环境变量指向`.boto`配置文件的位置的话,就会在更新WebRTC代码的中间`download_from_google_storage `时出现如下警告提示:

```
________ running 'download_from_google_storage --no_resume --platform=darwin --no_auth --bucket chromium-gn -s src/buildtools/mac/gn.sha1' in '/Users/apple/opensource/webrtc_build/webrtc2016/src/chromium'
NOTICE: You have PROXY values set in your environment, but gsutil in depot_tools does not (yet) obey them.
Also, --no_auth prevents the normal BOTO_CONFIG environment variable from being used.
To use a proxy in this situation, please supply those settings in a .boto file pointed to by the NO_AUTH_BOTO_CONFIG environment var.
```



####6. socks转http代理的其他玩法

```
http_proxy=http://localhost:8123 apt-get update

http_proxy=http://localhost:8123 curl www.google.com

http_proxy=http://localhost:8123 wget www.google.com

git config --global http.proxy 127.0.0.1:8123
git clone https://github.com/xxx/xxx.git
git xxx
git xxx
git config --global --unset-all http.proxy
```
