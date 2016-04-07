---
layout: post
category: 原创
tagline: "Supporting tagline"
tags: [iOS,WebRTC,AppRTC,stun,turn,ice]
---

##0.前言动机
早在去年初(2015年2月)的时候,戴维营教育由于课程需要讲WebRTC实时音视频聊天技术,就写过一个教程一步一步搭建一个WebRTC的后台服务器AppRTC的教程,但是由于Goggle官方代码有改变,导致大部分网友`严格`按步骤来操作不成功.

现在我们戴维营教育仍然要讲WebRTC技术,同时需要更新技术,在这里再一步一步重新搭建最新代码版本的AppRTC服务器,顺便整理出每一步的更新教程.
> 此教程于2016年3月18日更新版本
>更多原理介绍请参考戴维营教育2015年2月的旧教程:
>[http://io.diveinedu.com/2015/02/05/第六章-Webrtc服务器搭建.html](http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html)


##1.环境准备和代码下载

在Ubuntu Linux 版本服务器上创建新用户`apprtc`,设置密码,并安装必要的软件包:

```bash
$ sudo useradd -m apprtc
$ sudo passwd apprtc
$ sudo apt-get install openjdk-7-jdk 
$ sudo apt-get install python-webtest
```

然后在用户家目录下从Github克隆出`apprtc`代码仓库:

```bash
$ https://github.com/webrtc/apprtc.git apprtc_root
$ cd apprtc_root
$ git checkout fa43376fa2138a45cdcf14007695d2e7523ea48f
```
目前最新的版本是`fa43376fa2138a45cdcf14007695d2e7523ea48f`.
代码下载好了之后,就先留着不要去动, 下面我们去安装必要的依赖.


##2.安装NodeJS和grunt



```bash
$ cd $HOME;
$ wget https://nodejs.org/dist/v5.9.0/node-v5.9.0-sunos-x64.tar.xz
$ tar xvf node-v5.9.0-sunos-x64.tar.xz
$ export PATH=$PATH:$HOME/node-v5.9.0-linux-x64/bin
$ echo "export PATH=$PATH:$HOME/node-v5.9.0-linux-x64/bin" >> $HOME/.profile
$ sudo npm install -g npm
```
再安装grunt:

```bash
$ npm -g install grunt-cli
```



##3. 安装apprtc代码中的grunt依赖:

切换到apprtc用户下,终端进入apprtc仓库目录下:

执行下面命令:

```bash
apprtc@hostname:~/apprtc_root$ 
npm install
```

然后再执行下面命令编译出`apprtc`这个GAE app.

```bash
$ grunt build
```

编译完成后,输出将放在`~/apprtc_root/out`目录下的`app_engine`目录

当然到这一步还没完,还需要配置`constant.py`服务器参数,

```python
ICE_SERVER_BASE_URL = 'https://api.diveinedu.com'
ICE_SERVER_URL_TEMPLATE = '%s/apprtc/iceconfig.php?key=%s'
ICE_SERVER_API_KEY = os.environ.get('ICE_SERVER_API_KEY')
```


然后把下面命令放到一个脚本文件`$HOME/start_apprtc.sh`中:

```bash
export PATH=$PATH:$HOME/google_appengine
export APPRTC_APP=$HOME/apprtc_root/out/app_engine/
export HOST="--host=0.0.0.0"
export ICE_SERVER_API_KEY="AIzaSyAJdh2HkajseEIltlZ3SIXO02Tze9sO3NY"
dev_appserver.py $HOST $APPRTC_APP
```

以后都只需要执行该脚本文件就可以启动会话房间服务:

```bash
$ $HOME/start_apprtc.sh
```



打开浏览器,服务器服务器[http://apprtc.diveinedu.com:8080](http://apprtc.diveinedu.com:8080) ,就可以打开了.

最后一步就是配置Nginx反向代理服务器,提供默认HTTPS的访问, 新建Nginx虚拟主机配置文件,反向代理到8080端口.

配置文件位置是: `/etc/nginx/sites-enabled/apprtc.diveinedu.com`;
配置文件内容是: 

```nginx
#/etc/nginx/sites-enabled/apprtc.diveinedu.com
upstream roomserver {
        server localhost:8080;
}


server {
        listen 80 ;
        server_name apprtc.diveinedu.com;
        return  301 https://$server_name$request_uri;
}

server {
        listen 443 ;
        ssl on;
        # 域名为apprtc.diveinedu.com的SSL证书文件
        ssl_certificate      /etc/nginx/apprtc.diveinedu.com.crt;
        ssl_certificate_key  /etc/nginx/apprtc.diveinedu.com.key;

        server_name apprtc.diveinedu.com;
        access_log  /var/log/nginx/apprtc.diveinedu.com.log;
        location / {
                proxy_pass http://roomserver$request_uri;
                proxy_set_header Host $host;
        }

}
```

保存退出后, 重启nginx服务器.就可以通过`https://apprtc.diveinedu.com`安全连接方式访问会话房间服务器了.

关于免费的HTTPS/SSL证书请去沃通官网申请:
[https://buy.wosign.com/free/](https://buy.wosign.com/free/)


信令服务器还没准备好,下面再慢慢的来安装配置信令服务器.





##4. 安装GO环境

下载Go安装包:
    
```bash
    $ wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
    $ tar xvf go1.5.3.linux-amd64.tar.gz
```

创建本地GO代码目录:
    
```bash
$ mkdir -p $HOME/gopath/src
```

编辑打开文件`$HOME/.profile`,在文件末尾添加两行

```bash
$ export GOROOT=$HOME/go
$ export GOPATH=$HOME/gopath    
$ export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

退出终端重新打开或者登陆,GO环境就安装配置好了.

##5. 安装配置信令服务器

把AppRTC代码仓库内包含的GO语言信令服务器代码链接到`$GOROOT/src`下:

```bash
$ ln -rs $HOME/apprtc_root/src/collider/collider $GOPATH/src/
$ ln -rs $HOME/apprtc_root/src/collider/collidermain $GOPATH/src/
$ ln -rs $HOME/apprtc_root/src/collider/collidertest $GOPATH/src/
```

编辑`$GOPATH/src/collidermain/main.go`,修改房间服务器为我们前面的房间服务器:
    
```go
//var roomSrv = flag.String("room-server", "https://apprtc.appspot.com", "The origin of the room server")
var roomSrv = flag.String("room-server", "https://apprtc.diveinedu.com", "The origin of the room server")    
```

编辑`$GOPATH/src/collider/collider.go`,设置信令服务器所需要用的HTTPS的证书文件, 找到如下代码,注释后改为这样:

```go
//e = server.ListenAndServeTLS("/cert/cert.pem", "/cert/key.pem")
e = server.ListenAndServeTLS("/etc/nginx/apprtc.diveinedu.com.crt", "/etc/nginx/apprtc.diveinedu.com.key")
```




##6. 安装信令服务器collidermain的依赖

```bash
$ go get collidermain
```

安装GO环境的websocket包.
如果上面命令失败,那么则用下面这个麻烦的方法:

```bash
$ cd $GOPATH/src
$ wget http://www.golangtc.com/static/download/packages/golang.org.x.net.tar.gz
$ tar xvf golang.org.x.net.tar.gz
$ go install golang.org/x/net/websocket/
$ cd -
```

因为国内网络不好,不能自动下载包安装的话,就手动下载一个,手动安装这个包.
    
依赖项安装完后,我们安装`collidermain`命令服务器程序:


```bash
$ go install collidermain
```    

##7. 运行信令服务器

```bash
$  $GOPATH/bin/collidermain -port=8089 -tls=true
```

这样,信令服务器也可以通过安全链接方式来访问了.






##8. 整体测试

可以用`screen`来开启两个终端会话,分别来运行`AppRTC`会话房间服务器和`Collider`信令服务器.具体方式? 这就不赘述了.


到这一步,我们的AppRTC后台就完美搭建成功了,用谷歌火狐浏览器打开
[https://apprtc.diveinedu.com/](https://apprtc.diveinedu.com/r/12346678) 就可以随意的建立WebRTC视频通话了.


















