---
layout: post
category: 原创
tagline: "Supporting tagline"
tags: [iOS,AFNetworking,HTTPS,Openssl,Nginx]
---

###摘要
1.介绍iOS平台用AFNetworking与HTTPS后台接口进行安全通信。
2.介绍后台自签名证书制作步骤。
3.以及Linux平台Nginx配置HTTPS协议接口的Web站点。


这个关于AFNetworking的HTTPS安全通信的问题，很多没有过第一次经验的以及甚至有过一次经验的都会有点不确定。
其实很简单：
- A.对于后台服务器所配置动证书如果是经过CA机构认证颁发的，那么用户用AFNetworking来访问后台接口完全无感觉，就和http一样的方式。
- B.但是一个HTTPS的证书如果是知名CA机构认证颁发的，那么就会有问题，AFNetworking默认拒绝和这样的后台服务器通信，因为验证通不过，就和大家网页打开12306网站抢票一样，那个证书也不是经过CA颁发的，而是铁道部自己签名的一个证书。所以，对于中小型初创或是成长型公司来说，买一个https的证书也是需要花费不少费用的。所以大家在做后台通信的时候一般都自签名一个证书来实现https接口。自己签名的的证书可以用下面这个openssl命令进行生成：

```
openssl req -new -x509 -nodes -days 365 -newkey rsa:1024  -out tv.diveinedu.com.crt -keyout tv.diveinedu.com.key

```
其中:
-days 365是指定证书的有效期时间长度，单位是天，从命令运行的时刻算起；
-newkey rsa:1024是指定新生成的证书使用1024位长度的RSA非对称加密算法；
-out 指定输出的证书文件名
-keyout 指定输出的私钥文件名
上面这个命令运行后会要输入一些设置信息：

```
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Hunan
Locality Name (eg, city) :Changsha
Organization Name (eg, company) [Internet Widgits Pty Ltd]:tv.diveinedu.com
Organizational Unit Name (eg, section) :Market
Common Name (e.g. server FQDN or YOUR name) :tv.diveinedu.com
Email Address :diveinedu@qq.com

```
 
如果对搭建Linux后台HTTPS服务有兴趣，需要把证书和私钥上传到服务器或者直接在服务器生成，把此证书配置到后台服务器中，以Nginx为例进行如下设置:
- 1.先新增一个Nginx的虚拟主机配置文件，

```
sudo touch /etc/nginx/sites-available/tv.diveinedu.com
```

- 2.然后使这个配置文件生效：

```
sudo ln -sf /etc/nginx/sites-available/tv.diveinedu.com /etc/nginx/sites-enabled/tv.diveinedu.com
```

- 3.编辑该文件：

```
sudo vim /etc/nginx/sites-enabled/tv.diveinedu.com

```

- 4.敲入 i  进入VIM编辑模式，输入这样配置：

```

server {
	listen 80;#HTTP默认端口80
	server_name tv.diveinedu.com;#主机名,与HTTP请求头域的HOST匹配
	access_log  /var/log/nginx/tv.diveinedu.com.log;#访问日志路径
	return 301 https://$server_name$request_uri;#强制把所有http访问跳转到https
}

server {
	listen 443;#HTTPS默认端口443
	ssl on;#打开SSL安全Socket
	ssl_certificate      /etc/nginx/tv.diveinedu.com.crt;#证书文件路径
	ssl_certificate_key  /etc/nginx/tv.diveinedu.com.key;#私钥文件路径

	server_name tv.diveinedu.com;#主机名,与HTTP请求头域的HOST匹配
	access_log  /var/log/nginx/tv.diveinedu.com.log;#访问日志路径
	location / {
		root /var/www/tv.diveinedu.com/;#网站文档根目录
		index index.php index.html;#默认首页
	}
}

```

- 5.敲ESC后退出VIM的编辑模式，再敲入  x  回车  在Vim保存退出。
然后执行Nginx配置文件语法检查命令检查配置是否有错：

```
nginx -t

```

如果没有错误就会输出：

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```
- 6.然后就只需要重启Nginx服务器了

```
sudo service nginx restart

```

然后就去你的域名服务商后台把你的域名解析到服务器到IP地址就可以自由访问了，只不过会浏览器访问会被自动组织并显示警告，手动添加到信任即可。
 
如果公司有钱想为用户提供更好的服务和体验，最好还是去知名CA认证机构去注册申请一个有效的证书为妙！
不然浏览器(Chome)会这样：


![输入图片说明](https://static.oschina.net/uploads/img/201511/27120143_7hvl.png "我们刚刚的自签名证书")

![输入图片说明](https://static.oschina.net/uploads/img/201511/27134919_OW6L.png "12306的自签名证书"")
