---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [Metal, 模型, 3D, Swift, Objective-C]
---
##Metal第一课：清屏
这是Metal系列教程的第一课，主要介绍如何用Metal框架清除屏幕，后面的课程都基于本文之上，带我们一起熟悉3D渲染和图像处理基础。

注意，目前Metal还不支持iOS模拟器，因此你需要一台A7处理器的iOS设备（如iPhone 5s或retinal屏的iPad mini等）来运行下面的代码。

####创建新工程
下面我们使用Xcode的*Single View Application*模版创建项目。它会替我们创建一个视图控制器并设置在窗口上。
![](mages/xcode_ios_templates.png)

在“Build Phases > Link Binary with Libraries“中添加对Metal.framework和QuartzCore.frame的支持。
![](/images/add_metal_framework.png)

####UIKit界面
UIView使用Core Animation中的CALayer对象来处理实际的屏幕绘制操作。我们可以通过重载UIView的`+layerClass`方法改变UIView所使用的CALayer对象的类型。下面我们创建一个自定义的MetalView类（File > New > File...）。
![](/images/add_custom_view.png)

在MetalView.swift中引入Metal和QuartzCore。

```swift
import Metal
import QuartzCore
```

CAMetaLayer是由Core Animation框架而不是Metal框架提供的。通过使用CAMetalLayer可以绑定UIKit和Metal框架，并且为我们提供了一些有意思的功能。下面我们实现一下`+layerClass`方法。

```swift
import UIKit
import Metal
import QuartzCore

class MetalView : UIView {
    
    override class func layerClass (-> AnyClass {
        return CAMetalLayer.self
    }

}
```

```objc
#import <UIKit/UIKit.h>
#import <Metal/Metal.h>
#import <QuartzCore/QuartzCore>

@implementation MetalView
+ () layerClass
{
	return [CAMetalLayer class];s
}
@end
```

将Storyboard中的视图所用的类修改为MetalView，这样在程序启动时就会创建MetalView对象。为了方便起见，可以在MetalView中添加一个CAMetalLayer的属性，并在`-init`方法中将它赋值为`[self layer]`。这样可以避免以后在使用layer属性是总是需要强制类型转换。

编译运行，我们可以在设备上看到一个白色的屏幕。下面我们来学习Metal相关的设备和其它与实际绘制相关的对象。我们首先来看一下Metal所使用的协议。

####协议
Metal主要是通过协议来暴露它的功能的。许多Metal的API通过特定的协议返回对象。这样做的好处是我们不需要关心实际类的实现。下面是引用一个实现了`MTLDevice`协议的对象的语法。

```swift
var device: MTLDevice!
```

```objc
id <MTLDevice> device;
```

接下来我们尝试去获取和使用一个设备。
####设备
Metal设备是对GPU的一个抽象。它提供了创建命令队列、渲染状态、库等对象。我们将会一一学习它们。Metal提供了一个C语言的函数`MTLCreateSystemDefaultDevice`来创建和返回一个`MTLDevice`类型的设备来满足我们的需求。这个函数不需要任何参数，因为所有的设备属性都不需要我们设置。

Metal layer需要知道哪个设备将会被渲染到它里面。通过设置layer的颜色格式来确定每个颜色分量的大小和顺序，我们选择`MTLPixelFormatBGRA8Unorm`，每个像素都由蓝、绿、红以及Alpha值组成，每个分量为8位无符号整数（0-255）。

可以在MetalView中创建一个设备属性，因为后面我们会使用它来创建各种资源。下面是`-init`方法的设置代码。

```swift
var _metalLayer: CAMetalLayer!
var _device: MTLDevice!

required init(der aDecoder: NSCoder) {
    super.init(der: aDecoder)

    _metalLayer = self.layer as CAMetalLayer
    _device = MTLCreateSystemDefaultDevice()
    _metalLayer.device = _device;
    _metalLayer.pixelFormat = MTLPixelFormat.BGRA8Unorm
}
```

```objc
- (stancetype)initWithCoder:(Coder *)aDecoder
{
	if (elf = [super initWithCoder: aDecoder]))
    {
    	_metalLayer = (MetalLayer *)[self layer];
        _device = MTLCreateSystemDefaultDevice(
        _metalLayer.device = _device;
        _metalLayer.pixelFormat = MTLPixelFormatBGRAUnorm;
    }
    
    return self;
}
```

####`redraw`方法
在后面的文章中我们会在`-redraw`方法中添加绘制命令，但是现在只是使用一种固定的颜色来清理屏幕，因此只需要调用`-redraw`方法一次。我们通过重写`-didMoveToWindow`方法来进行绘制。

```swift
override func didMoveToWindow({
    self.redraw()
}
```

```objc
- (id)didMoveToWindow
{
	[self redraw];
}
```

redraw方法完成所有的清理工作，因此下面部分的代码全部是在redraw中完成的。

####纹理与贴图
Metal中的纹理是一个图片的集合。Metal运行用一个纹理对象表示一个图片数组，其中每个元素都叫做一个切片。纹理中的图片都有一个特定的尺寸和像素格式。Metal中的纹理可以是1D、2D和3D的。当然，我们现在只需要一个2D纹理作为帧缓存，这个纹理与设备的分辨率相同。我们使用*Core Animation*中的`CAMetalDrawable`协议获取该纹理的引用。

drawable对象是对纹理的一个简单封装。每次进行绘制的时候，我们都可以从Metal layer获取一个drawable对象，并从中得到一个纹理。

```swift
let drawable = _metalLayer.nextDrawable()
let texture = drawable.texture
```

```objc
id<CAMetalDrawable> drawable = [self.metalLayer nextDrawable];
id<MTLTexture> texture = drawable.texture;
```

当我们将内容渲染到纹理上以后，同样是通过drawable对象给*Core Animation*发送信号来呈现在屏幕上。为了真正的清除帧缓存，我们需要设置一个渲染描述符来描述每一帧的动作。

####渲染描述符
*渲染描述符*通知Metal如何渲染一副图像。在渲染过程的开始，`locadAction`决定是否保留之前的材质内容。`storeAction`决定何种渲染效果作用在材质上。由于我们想要在整个屏幕上绘制像素点，将store action设置为`MTLStoreActionStore`。

渲染描述符也是我们设置清屏颜色的地方。下面使用不透明的红色作为清除色（red = 1, green = 0, blue = 0, alpha = 1）。

```swift
let passDescriptor = MTLRenderPassDescriptor()
passDescriptor.colorAttachments[0].texture = texture;
passDescriptor.colorAttachments[0].loadAction = .Load
passDescriptor.colorAttachments[0].storeAction = .Store
passDescriptor.colorAttachments[0].clearColor = MTLClearColorMake(0, 0.0, 0.0, 1.0)
```

```objc
MTLRenderPassDescriptor *passDescriptor = [MTLRenderPassDescriptor renderPassDescriptor];
passDescriptor.colorAttachments[0].texture = texture;
passDescriptor.colorAttachments[0].loadAction = MTLLoadActionClear;
passDescriptor.colorAttachments[0].storeAction = MTLStoreActionStore;
passDescriptor.colorAttachments[0].clearColor = MTLClearColorMake(0, 0.0, 0.0, 1.0);
```

渲染描述符用来配置渲染命令。

####队列、缓存和编码器
*command queue*是一个保存即将执行的渲染命令缓存列表的对象。我们可以通过设备获取一个命令队列。一般情况下，命令队列时一个长期存在的对象，因此在一些高级应用场景，我们需要保留这个队列不只一帧的时间。

```swift
let commandQueue = _device.newCommandQueue()
```

```objc
id<MTLCommandQueue> commandQueue = [self.device newCommandQueue];
```

*command buffer*将一组即将执行的渲染命令表示为一个单元。每个命令缓存都关联着一个命令队列。

```swift
let commandBuffer = commandQueue.commandBuffer()
```

```objc
id<MTLCommandBuffer> commandBuffer = [commandQueue commandBuffer];
```

*command encoder*是一个用来告诉Metal如何绘制的对象。它负责将这些上层命令（设置shader参数、绘制三角形等）转化为对应的底层指令，然后写入相应的命令缓存。一旦完成所有的绘制调用，给编码器发送`endEncoding`消息就可以结束编码。

```swift
let commandEncoder = commandBuffer.renderCommandEncoderWithDescriptor(ssDescriptor)
commandEncoder?.endEncoding()
```

```objc
id<MTLRenderCommandEncoder> commandEncoder = [commandBuffer renderCommandEncoderWithDescriptor: passDescriptor];
[commandEncoder endEncoding];
```

最后，一旦前面所有的命令都完成了，命令缓存就会发送绘制信号。我们通过调用`commit`来标明这个命令缓存意见完成，可以放入命令队列中供GPU执行。这样我们的帧缓存就会被红色填满。

```swift
commandBuffer.presentDrawable(awable);
commandBuffer.commit()
```

```objc
[commandBuffer presentDrawable: drawable];
[commandBuffer commit];
```

####编译运行
编译运行就可以看到效果了。当然，Metal应用目前还只能在真机上运行。
![](/images/clear_screen_result.png)

```swift
//
//  MetalView.swift
//  ClearScreen
//
//  Created by WuQiong on 14/12/15.
//  Copyright ( 2014年 戴维营教育. All rights reserved.
//

import UIKit
import Metal
import QuartzCore

class MetalView: UIView {
    var _metalLayer: CAMetalLayer!
    var _device: MTLDevice!
    
    required init(der aDecoder: NSCoder) {
        super.init(der: aDecoder)
        
        _metalLayer = self.layer as CAMetalLayer
        _device = MTLCreateSystemDefaultDevice()
        _metalLayer.device = _device;
        _metalLayer.pixelFormat = .BGRA8Unorm
    }
    
    override class func layerClass (-> AnyClass {
        return CAMetalLayer.self
    }

    override func didMoveToWindow({
        self.redraw(
    }
    
    func redraw({
        let drawable = _metalLayer.nextDrawable(
        let texture = drawable.texture;
        
        let passDescriptor = MTLRenderPassDescriptor()
        passDescriptor.colorAttachments[0].texture = texture;
        passDescriptor.colorAttachments[0].loadAction = .Clear
        passDescriptor.colorAttachments[0].storeAction = .Store
        passDescriptor.colorAttachments[0].clearColor = MTLClearColorMake(0, 0.0, 0.0, 1.0)
        
        let commandQueue = _device.newCommandQueue()
        let commandBuffer = commandQueue.commandBuffer()
        
        let commandEncoder = commandBuffer.renderCommandEncoderWithDescriptor(ssDescriptor)
        commandEncoder?.endEncoding()
        
        commandBuffer.presentDrawable(awable);
        commandBuffer.commit()
    }
}
```

源码下载地址：
[ClearScreen.zip](/Example/ClearScreen.zip)

原文地址：
[](http://metalbyexample.com/up-and-running-1/)

--------
长沙戴维营教育iOS开发培训，长沙iOS培训的摇篮
[http://www.diveinedu.cn](http://www.diveinedu.cn)
长沙岳麓山国家大学科技园
