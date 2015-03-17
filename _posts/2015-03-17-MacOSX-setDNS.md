---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [MacOSX, DNS, SystemConfiguration, makefile]
---
通过C语言接口在Mac App内部对系统的DNS配置进行修改。

#Mac OS X设置DNS代码

示例代码setDNS.c内容如下：

```c
#include <SystemConfiguration/SystemConfiguration.h>

static bool setDNS(CFStringRef *resolvers, CFIndex resolvers_count)
{
	SCDynamicStoreRef ds = SCDynamicStoreCreate(NULL, CFSTR("setDNS"), NULL, NULL);

	CFArrayRef array = CFArrayCreate(NULL, (const void **) resolvers,
			resolvers_count, &kCFTypeArrayCallBacks);

	CFDictionaryRef dict = CFDictionaryCreate(NULL,
			(const void **) (CFStringRef []) { CFSTR("ServerAddresses") },
			(const void **) &array, 1, &kCFTypeDictionaryKeyCallBacks,
			&kCFTypeDictionaryValueCallBacks);    

	CFArrayRef list = SCDynamicStoreCopyKeyList(ds,
			CFSTR("State:/Network/(Service/.+|Global)/DNS"));

	CFIndex i = 0, j = CFArrayGetCount(list);
	if (j <= 0) {
		return FALSE;
	}
	bool ret = TRUE;
	while (i < j) {
		ret &= SCDynamicStoreSetValue(ds, CFArrayGetValueAtIndex(list, i), dict);
		i++;
	}
	return ret;
}

int main(int argc, const char * argv[])
{
	CFStringRef resolvers[] = {
		CFSTR("8.8.8.8"),
		CFSTR("114.114.114.114")
	};
	setDNS(resolvers, (CFIndex) (sizeof resolvers / sizeof resolvers[0]));

	return 0;
}

```

对应的Makefile文件内容：

```c
#!/usr/bin/make -f
default: setDNS.c
        cc -o setDNS setDNS.c -framework Foundation -framework SystemConfiguration
clean:
        rm setDNS
```



----
谢谢各位，欢迎交流并指正。

----  大茶园丁@戴维营教育

http://io.diveinedu.com

http://www.diveinedu.com

https://github.com/DiveinEdu-CN

![](/images/qrcode-diveinedu-mp-weixin.jpg)


