---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [jQuery, Javascript, HTML 5]
---
####`this`关键字的使用
在JavaScript中使用`this`关键字表示调用方法的对象，这与大部分面向对象语言是一样的。但是由于`call`、`apply`、`bind`等函数的影响，我们可以改变`this`所代指的对象。

- 使用`call`或者`apply`调用的函数中，`this`代指传入的第一个参数对象，如果传入`null`或者`undefined`，则表示全局对象(`window`)。
- 通过对象调用函数（方法），函数中的`this`表示调用该函数的对象。
- 单独调用的函数中`this`表示全局对象。

```js
var myObject = {
	sayHello: function() {
    	console.log("Hi, my name is " + this.myName);
    },
    myName: "Rebecca"
};

var secondObject = {
	myName: "Colin"
};

myObject.sayHello();	//"Hi, my name is Rebecca"
myObject.sayHello.call(secondObject);	//"Hi, my name is Colin"
```

```js
var myName = "the global object";
var sayHello = function() {
	console.log("Hi, my name is " + this.myName);
};
var myObject = {
	myName = "Rebecca"
};
var myObjectHello = sayHello.bind(myObject);

sayHello();	//"Hi, my name is the global object"
myObjectHello(); //"Hi, my name is Rebecca"
```

JavaScript可以在运行中为对象动态添加函数，这样也会导致`this`代指的对象发生变化。

```js
var myName = "the global object";
var sayHello = function() {
	console.log("Hi, my name is " + this.myName);
};
var myObject = {
	myName: "Rebecca"
};
var secondObject = {
	myName: "Colin"
};

myObject.sayHello = sayHello;
secondObject.sayHello = sayHello;

sayHello();	//"Hi, my name is the global object"
myObject.sayHello(); //"Hi, my name is Rebecca"
secondObject.sayHello(); //"Hi, my name is Colin"
```

还有，不能直接给处于较深的名字空间的函数增加短引用，会导致`this`变为全局对象，但是可以对它所处的对象设置短引用。

```js
var myNamespace = {
	myObject: {
    	sayHello: function() {
        	console.log("Hi, my name is " + this.name);
        },
        myName: "Rebecca"
    }
};

var hello ＝ myNamespace.myObject.sayHello;
hello();	//"Hi, my name is undefined"

var obj = myNamespace.myObject;
obj.sayHello();	//"Hi, my name is Rebecca"
```

**Tips**
> `call`与`apply`的区别是，`apply`接收两个参数：`this`和函数的参数数组；而`call`的第一个参数为`this`，但是后面是函数的参数列表。

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。