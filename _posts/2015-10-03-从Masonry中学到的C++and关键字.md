---
layout: post
category: 原创
tagline: "Supporting tagline"
tags: [Masonry, Objective-C, 自动布局, C++]
---

一直在用Masonry对视图进行自动布局（AutoLayout），它的串联设置的方式很方便，多个相同的约束条件可以用`and`进行连接。今天**[@大茶园丁](https://github.com/wuqiong)**在测试[**MobileVLCKit**](https://github.com/wuqiong/MobileVLCKit-SDK)时突然发现`and`变红了，貌似是一个关键字，并且编译也通不过。于是百度确认了一下，结果在[@余璜](http://blog.csdn.net/maray/article/details/41644235)同学的博客上早就介绍了（顺便汗颜了一下，一直说的坚持写博客）。

```objc
[v mas_makeConstraints:^(MASConstraintMaker *make) {
    v.left.and.right.equalTo(@20);
}];
```

####一、C语言
其实在C语言中就已经定义了许多宏，用来增加一些运算符的可读性，只是用的人比较少（**iso646.h**）。我们只要包含这个头文件就可以使用`and`、`or`之类的作为运算符了。

```cpp
#ifndef __ISO646_H
#define __ISO646_H

#ifndef __cplusplus
#define and    &&
#define and_eq &=
#define bitand &
#define bitor  |
#define compl  ~
#define not    !
#define not_eq !=
#define or     ||
#define or_eq  |=
#define xor    ^
#define xor_eq ^=
#endif

#endif /* __ISO646_H */
```

```cpp
#include <stdio.h>
#include <iso646.h>

int main(int argc, char *argv[]) {
	int a = 0;
	int b = 1;
    //使用and代替&&
	if(a and b) {
		printf("真\n");
	}
	else {
		printf("假\n");
	}
	return 0;
}
```

直接用clang进行编译。

```bash
$ clang keywords.c -o keyword
$ ./keyword
假
```

####二、C++
不过在C++中就更简单了，直接成为了关键字，使用它们的时候不需要添加任何头文件。

```cpp
#include <iostream.h>

int main(int argc, char *argv[]) {
	int a = 0;
	int b = 1;
    //使用and代替&&
	if(a and b) {
		std::cout << "真" << std::endl;
	}
	else {
		std::cout << "假" << std::end;
	}
	return 0;
}
```

```bash
$ clang keywords.cpp -o keyword
$ ./keyword
假
```

####参考资料
1. [https://zh.wikipedia.org/wiki/C替代标记](https://zh.wikipedia.org/wiki/C替代标记)