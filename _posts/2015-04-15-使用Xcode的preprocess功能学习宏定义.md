---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [Xcode, Objective-C, 宏定义, 编译预处理]
---

C、C\+\+和Objective-C都支持宏定义。宏在编译预处理过程中会进行代码替换。我们可以通过命令行非常容易就能的到一个源文件处理后的结果。

```objc
#define kMax 100

int main(int argc, char * argv[]) {
    int a = 100;
    if (a < kMax) {
        a++;
    }
    else {
        a = 0;
    }
    
    return 0;
}
```

编译预处理命令：

```bash
clang -E main.m
```

预处理后的结果：

```objc
int main(int argc, char * argv[]) {
    int a = 100;
    if (a < 100) {
        a++;
    }
    else {
        a = 0;
    }

    return 0;
}
```

当然，在实际应用中还有许多其它的预处理指令，包括`include`、`if`、`ifdef`等等。并且还有一些符号，比如连接`##`、转化字符串`#`等。并且还可以定义函数形式的宏，以及在宏定义中引用其它宏。这样导致初学者很难把握到底一个宏展开后是什么样子的。虽然我们可以通过命令行进行处理，但终归不方便。实际上从Xcode 4开始就有预处理和汇编的功能，能够非常方便的查看一个源文件的预处理结果或者汇编代码。

#####两种方式查看处理结果

1. 使用菜单
**Product > Perform Action > Preprocess "xxx"**

2. 使用辅助编辑器
打开辅助编辑器后，编辑编辑器上面选择文件的地方，从弹出菜单中执行**Preprocess**或**Assembly**功能。

![](/images/objc/xcode_preprocess_assistant.png)

参考资料：
1. [Xcode预处理功能：https://bpoplauschi.wordpress.com/2014/02/20/xcode-preprocess-assistant/](https://bpoplauschi.wordpress.com/2014/02/20/xcode-preprocess-assistant/)
2. [gcc宏定义Macro：https://gcc.gnu.org/onlinedocs/cpp/Macros.html](https://gcc.gnu.org/onlinedocs/cpp/Macros.html)
3. [微软宏定义：https://msdn.microsoft.com/en-us/library/503x3e3s.aspx](https://msdn.microsoft.com/en-us/library/503x3e3s.aspx)
4. [常用宏定义：http://www.cplusplus.com/doc/tutorial/preprocessor/](http://www.cplusplus.com/doc/tutorial/preprocessor/)
> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

