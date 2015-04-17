---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [设计模式, 外观模式, Facade, Objective-C]
---

一个框架中如果包含的类比较多，或者功能比较复杂的情况下，可以通过一个较辅助类为一些常用的情况提供简单的接口。这样客户在使用这个框架的时候既可以比较简单的应付常见的场景，又可以使用框架中的内实现符合自己要求的功能。这就好比买电脑的时候，即可以买品牌机，又可以自己买配件组装。下面就拿买电脑来举例。

假设电脑由显示器、主板、CPU、内存和显卡组成。当然，实际远远不止这些。每个设备都有许多的参数需要选择，我们给它们分别定义一个类去完成选择的工作。

```objc
//选择显示器
@interface Display : NSObject
- (void)chooseDisplay:(NSString *)type;
@end

@implementation Display
- (void)chooseDisplay:(NSString *)type {
	NSLog(@"选择显示器：%@", type);
}
@end

//选择主板
@interface MainBoard : NSObject
- (void)chooseMainBoard:(NSString *)type;
@end

@implementation MainBoard
- (void)chooseMainBoard:(NSString *)type {
	NSLog(@"选择主板：%@", type);
}
@end

//选择CPU
@interface CPU : NSObject
- (void)chooseCPU:(NSString *)type;
@end

@implementation CPU
- (void)chooseCPU:(NSString *)type {
	NSLog(@"选择CPU：%@", type);
}
@end

//客户代码
@interface Client : NSObject
- (void)assebleComputer;
@end

@implementation Client 
- (void)assebleComputer {
	Display *display = [Display new];
    [display chooseDisplay: @"AOC"];
    
    MainBoard *mainBoard = [MainBoard new];
    [mainBoard chooseMainBoard: @"华硕"];
    
    CPU *cpu = [CPU new];
    [cpu chooseCPU: @"i7处理器"]
}
@end
```

这时的输出结果应该是：
```bash
选择显示器：AOC
选择主板：华硕
选择CPU：i7处理器
```

类图：
![](/images/pattern/no_facade.png)

我们可以利用这几个选取的类去组装任意配置的电脑。但是如果遇到小白用户，他们可能从来没有听说过什么是处理器，什么是主板。这时我们可能希望有一个简单一点地方式去做这件事。

```objc
@interface MacBookPro : NSObject
- (void)chooseDevice;
@end

@implementation MacBookPro
- (void)chooseDevice {
	Display *display = [Display new];
    [display chooseDisplay: @"三星"];
    
    MainBoard *mainBoard = [MainBoard new];
    [mainBoard chooseMainBoard: @"华硕"];
    
    CPU *cpu = [CPU new];
    [cpu chooseCPU: @"i5处理器"]
}
@end
```

这时客户的代码为：
```objc
@interface Client : NSObject
- (void)assebleComputer;
@end

@implementation Client
- (void)assembleComputer {
	MacBookPro *computer = [MacBookPro new];
    [computer chooseDevice];
}
@end
```

输出结果为：
```bash
选择显示器：三星
选择主板：华硕
选择CPU：i5处理器
```

使用`MacBookPro`这个类的好处就是客户可以不去了解电脑的组成的情况下，也可以得到一台完整的电脑。这样就可以同时满足小白用户和专业用户了。下面是它们之间的类图。
![](/images/pattern/facade.png)

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

