---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, CocoaPods, 依赖管理工具]
---
####什么是远程消息推送功能
苹果给iOS和Mac添加了消息推送的功能，使得我们可以通过后台服务器给应用程序（APP）发送消息，不管APP是否正在使用，比如邮箱的来件提示功能。这项服务被称为**Apple Push Notification service**（APNs）。里面一共涉及到四个角色：APP、设备、APNs和应用后台服务器（Provider），其中APP、后台服务器和APNs之间使用deviceToken唯一的标识一个用户。

![](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/registration_sequence_2x.png)

推送服务的工作流程：

1. APP向系统注册推送服务。
2. 设备从APNs请求deviceToken。
3. 通过代理方法将deviceToken返回给APP。
4. APP将deviceToken发送给应用后台服务器（Provider）。
5. 应用后台服务器保存deviceToken，然后在需要推送通知的时候，给APNs发送信息，使用deviceToken标识所要送达的客户端。
6. APNs将后台服务器发过来的数据推送到设备。
7. 设备将消息分发给应用程序。

在使用推送功能的时候，需要在开发者中心创建支持Push Notification的证书，并且将证书和私钥用于应用后台服务器与APNs之间通信。

####环境配置
使用推送服务有一些必要条件：

1. 开发者账号。
2. iOS真机（iPhone、iPad、iPod）。
3. 后台服务器。
4. 网络。

为了使应用支持推送服务，需要配置Provisioning Profile使它支持Push，和普通的Provisioning Profile文件一样分为Development和Production两个版本。我们使用Development版进行测试。

![](http://cdn5.raywenderlich.com/wp-content/uploads/2011/05/Screen-shot-2011-05-09-at-5.11.18-PM-250x87.jpg)

接下来创建一个用于应用后台服务器和APNs服务器通信时使用的SSL证书和私钥。

1 .在**钥匙串访问**工具中获取证书请求文件（CSR）。
 
![](http://cdn2.raywenderlich.com/wp-content/uploads/2011/05/Keychain-Access-1-Request-Certificate-500x200.jpg)

2 .保存请求文件。

![](http://cdn5.raywenderlich.com/wp-content/uploads/2013/04/Keychain-Access-2-Generate-CSR.jpg)

3 .从**钥匙串访问**工具中导出私钥，将它保存为**PushKey.p12**，输入密码**abcde**。千万别把密码给忘了哈，等下要用的。

![](http://cdn2.raywenderlich.com/wp-content/uploads/2011/05/Keychain-Access-3-Export-Private-Key-500x279.jpg)

4 .登陆**[iOS Dev Center](https://developer.apple.com/ios)**创建*APP ID*和＊Provisioning Profile*。

![](http://cdn3.raywenderlich.com/wp-content/uploads/2013/04/select_certificates_identifiers_profiles-480x269.png)

![](http://cdn5.raywenderlich.com/wp-content/uploads/2013/04/certificates_identifiers_profiles_section-480x286.png)

![](http://cdn3.raywenderlich.com/wp-content/uploads/2013/04/create_app_id-480x292.png)

5 .创建新的App ID时，要注意开启**Push Notification**。

![](http://cdn3.raywenderlich.com/wp-content/uploads/2013/04/registeration_complete-446x320.png)

6 .最后App ID看起来是这样的。

![](http://cdn2.raywenderlich.com/wp-content/uploads/2013/04/push_app_accordion-480x301.png)

7 .到这一步，虽然已经开启了推送服务，但是还需要进一步配置，点击**Setting**按钮进行设置。

![](http://cdn3.raywenderlich.com/wp-content/uploads/2013/04/app_id_settings-480x293.png)

8 .滚动到最下面，需要创建SSL证书（**Create Certificate**），测试环境使用**Development SSL Certificate**。

9 .查看证书创建步骤和说明，上传**第1步**得到的证书请求文件。

![](http://cdn2.raywenderlich.com/wp-content/uploads/2013/04/generate_dev_certificate-480x267.png)

10 .下载生成好的证书，命名为**aps_development.cer**。

![](http://cdn1.raywenderlich.com/wp-content/uploads/2013/04/click_download-480x159.png)



* * *

> 到了这里，我们有多种选择继续了。

1 .使用第三方小工具**[PushMeBaby](https://github.com/stefanhafeneger/PushMeBaby)**模拟应用后台服务器发送推送信息。

2 .搭建应用后台服务器发送推送信息。


*****

下面先试一试第一种方法，使用**PushMeBaby**。这是一个开源的Mac小程序，我们直接去Github下载源码，用Xcode打开，将**ApplicationDelegate.m**中天上deviceToken和证书的位置。

```objc
- (id)init {
	self = [super init];
	if(self != nil) {
    	//77e231f0 76257e00 eed93ac6 47b52c78 12bae79f 9c9d1c67 4c990589 36c9a235 ---- 保留空格
		self.deviceToken = @"";
        //推送内容，JSON格式
		self.payload = @"{\"aps\":{\"alert\":\"长沙戴维营教育iOS开发培训最好！\",\"badge\":1}}";
        //获取证书路径
		self.certificate = [[NSBundle mainBundle] pathForResource:@"aps_development" ofType:@"cer"];
	}
	return self;
}
```

>deviceToken的获取在下面的代码部分。

一定要记得将刚才的证书文件添加到项目中，当然也可以直接将证书路径赋值给self.certificate。


*****

接下来试一试第二种方法，我们使用PHP来发送通知（其实运行PHP并不需要另外搭建服务器和下载程序，Mac默认支持PHP运行，不信到命令行运行一下`php`）。

这种方式相对来说麻烦一些，但是也是实际使用的时候会采取的方式。我们需要进一步处理私钥和证书文件。

1 .首先将证书文件和私钥处理成单个方便使用的pem文件，假设CSR、p12和cer文件都放在桌面上。

```bash
$ cd ~/Desktop
$ls 
aps_development.cer
CertificateSigningRequest.certSigningRequest
PushKey.p12
```

2 .将aps_development.cer转换为pem文件。

```bash
$ openssl x509 -in aps_development.cer -inform der -out PushCert.pem
$ ls
aps_development.cer
CertificateSigningRequest.certSigningRequest
PushCert.pem
PushKey.p12
```

3 .将p12私钥文件转换为pem文件。

```bash
$ openssl pkcs12 -nocerts -out PushKey.pem -in PushKey.p12 
Enter Import Password:
MAC verified OK
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

4 .将两个文件合成同一个。

```bash
$ cat PushCert.pem PushKey.pem > ck.pem
$ ls
aps_development.cer
CertificateSigningRequest.certSigningRequest
ck.pem
PushCert.pem
PushKey.pem 
PushKey.p12
```

5 .测试证书是否有效。

```bash
$ openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert PushCert.pem -key PushKey.pem 
```

如果有效的话，会输出一堆信息，并且建立连接，否则不会成功建立连接。

6 .使用PHP进行测试，下载[SimplePush.php](http://d1xzuxjlafny7l.cloudfront.net/downloads/SimplePush.zip)，修改文件并填入deviceToken和密码。在终端运行该代码。

```bash
$ php simplepush.php 
Connected to APNS
Message successfully delivered
```

成功发送推送消息。

####代码实现
有了上面的这些准备工作，iOS端的开发非常简单，UIApplicaton中有好几个方法都与推送消息有关，包括本地推送。

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	//判断是否注册了远程通知
    if (![application isRegisteredForRemoteNotifications]) {
        UIUserNotificationSettings *uns = [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeAlert|UIUserNotificationTypeBadge|UIUserNotificationTypeSound) categories:nil];
        [application registerUserNotificationSettings:uns];
        //注册远程通知
        [application registerForRemoteNotifications];
    }
        
    return YES;
}

//注册成功，返回deviceToken
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"%@", deviceToken);
}

//注册失败
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
    NSLog(@"%@", error);
}

//接收到推送消息
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    NSLog(@"remote: %@", userInfo);
}
```

> 甚至可以使用APNs实现一个聊天工具，具体请查看参考资料（4）。

####参考资料

1. [https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW9](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW9)
2. [https://github.com/stefanhafeneger/PushMeBaby](https://github.com/stefanhafeneger/PushMeBaby)
3. [http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1](http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1)
4. [http://www.raywenderlich.com/32963/apple-push-notification-services-in-ios-6-tutorial-part-2](http://www.raywenderlich.com/32963/apple-push-notification-services-in-ios-6-tutorial-part-2)

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

