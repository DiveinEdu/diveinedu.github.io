---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [jQuery, Javascript, HTML 5]
---
> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

####注释
JavaScript支持与C语言相同的注释方法，单行注释(`//`)和多行注释（`/* */`）。代码中的注释在执行的时候将被忽略，只是起到说明代码的功能，便于代码维护和理解。

```js
//单行注释

/*
多行注释
1. 这是一行注释
2. 这还是一行注释
3. 注释只是起到说明性作用
*/
```

####空白字符
空白字符包括空格、回车、换行等符号，在JavaScript中也会被忽略掉，只是起到维持代码格式的作用，方便查看。在实际应用中，JavaScript代码部署到服务器上时常常使用工具将代码中的空白字符去掉，以便减少文件大小。

```js
//适当的空白字符可以增加代码的可读性
var foo = function() {
	for (var i = 0; i < 10; i++) {
    	alert(i);
    }
};
foo();

//等价
var foo=function(){for(var i=0;i<10;i++){alert(i);}};foo();
```

####保留字（关键字）
在定义标识符的时候不能使用关键字或保留字。

- ECMAScript 6的关键字

| break | extends |	switch	|
|-------|--------|---------|
| case  | finally |	this  |
| class | for | throw |
| catch | function | try |
| const | if | typeof |
| continue | import | var |
| debugger | in | void |
| default | instanceof | while |
| delete | let | with |
| do | new | yield |
| else | return | |
| export | super | |

- ECMAScript预留关键字(enum、await)
- strict模式语录关键字

| implements | static | public |
|------------|--------|--------|
|package|interface||
|protected|private||

- ECMAScript老规范中的关键字（ECMAScript 1-3）
|abstract|final|native|
|--------|-----|------|
|boolean|float|short|
|byte|goto|synchronized|
|char|int|transient|
|double|long|volatile|

- 字面常量`false`、`true`以及`null`也是保留字

####标识符
标识符作为变量或函数的名字，在后续的代码中可以引用它所表示的变量或函数。标识符的命名必须遵循以下规则：

- 不能与保留字（关键字）同名。
- 只能由字母、数字、`$`以及下划线`_`组成。
- 第一个字符不能为数字。

>标识符命名应该尽量使用有意义的名字。

```js
//合法的标识符
var myAwesomeVariable = "a";
var myAwesomeVariable2 = "b";
var my_awesome_variable = "c";
var $my_AwesomeVariable = "d";
var _my_awesome_variable_$ = "d";

//非法的标识符
var 345_abc = "a";	//不能以数字开头
var break = "b";	//不能使用保留字
var ***abc = "c";	//只能使用字母、下划线、$以及数字
```

####变量定义
JavaScript使用`var`定义变量。

```js
var test = 1;
var test2 = function() {};

var test3 = 2, test4 = function(){};
```

没有初始化的变量默认值为`undefined`。

```js
var x;
x === undefined;	//true
```

####数据类型
- 基本数据类型
a. 字符串（String），JavaScript中的字符串可以用单引号`'`和双引号`"`表示。

```js
var a = "I am a string";
var b = 'So am I!';
var c = 'I have "!'
var c = "I must use escape\"".	//使用转义字符
```

b. 数值类型，可以是任意的正数、负数，JavaScript没有明显的浮点数和整数之分。
c. 布尔类型，可以为`true`和`false`。
d. JavaScript中还有两个特殊的类型`null`和`undefined`。`null`表示没有值，这与C语言中的`NULL`类似。而`undefined`表示变量没有赋值。

```js
//null
var foo = null;

//undefined
var bar1 = undefined;
var bar2;
```

####对象
JavaScript中所有东西都是对象。我们可以在[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference#Global_Objects]上找到大量的内置对象。下面主要介绍Object、Array和Function。

使用字面对象是最简单的对象创建方法。对象是由无序的键值对组成，其中键被称为属性`property`，而值可以是任的JavaScript类型的对象。我们通过使用点语法和括号语法创建和访问一个对象的属性。

```js
//空对象
var person1 = {};

//使用点语法给属性赋值
person1.firstName = "John";
person1.lastName = "Doe";

//使用点语法访问属性
alert(person1.firstName + " " + person1.lastName);

//创建对象
var person2 = {
	firstName = "Jane",
    lastName = "Doe"
};

alert(person2.firstName + " " + person2.lastName);

var people = {};

//括号语法给属性赋值
people["person1"] = person1;
people["person2"] = person2;

//混合使用两种语法
alert(people["person1"].firstName);
alert(people["person2"].firstName);
```

>如果一个属性在访问的时候还没有定义，将返回`undefined`结果。

```js
var person = { name: "John Doe"};
alert( person.email ); 	//undefined
```

在JavaScript中，几乎所有东西都是对象，数组、函数、数值甚至字符串等都是对象，它们都有自己的方法和属性。

```js
var myObject = {
	sayHello: function() {
    	console.log("hello");
    },
    myName: "Rebecca"
};

myObject.sayHello();			//"hello"
console.log(myObject.myName);	//"Rebecca"
```

创建字母对象的时候，可以用任意的标识符、字符串或者是数值作为`key`。

####Array
数组是一个存放有序元素的对象。我们可以使用从0开始的下标操作数组中的元素，如果超出范围则返回`undefined`。每个数组对象都有一个`length`属性表示元素的个数。

```js
//使用构造器创建数组
var foo = new Array();

//使用字面方式创建对象
var bar = [];
```

这两种创建数组的方式都会将参数作为它的初始元素，但是需要注意的一点是：*使用构造器创建的时候，如果只有一个参数，构造器将认为这是数组的大小！*。

```js
//foo.length == 1
var foo = [ 100 ];
alert( foo[0] );		//100
alert( foo.length );	//1

//bar.length == 100
var bar = new Array(100);
alert(bar[0]);		//undefined
alert(bar.length);	//100
```

数组对象拥有一系列常用的函数。

```js
var foo = [];

foo.push("a");	//添加元素
foo.push("b");	//添加元素
foo[2] = "c";	//添加元素

alert(foo[0]);	//a
alert(foo[1]);	//b

alert(foo.length);	//c

foo.pop();

alert(foo[0]);	//a
alert(foo[2]);	//undefined

alert(foo.length);	//2

foo.unshift("z");

alert(foo[0]);	//z
alert(foo[1]);	//a

alert(foo.length);	//3

foo.shift();

alert(foo[0]);	//a
alert(foo[2]);	//undefined

alert(foo.length);	//2
```

*Array的方法和属性*
- `.length`获取数组中元素个数。

```js
var myArray = ["Hello", "world", "!"];
console.log(myArray.length);	//3
```

- `.concat()`连接两个数组。

```js
var myArray = [2, 3, 4];
var myOtherArray = [5, 6, 7];
var wholeArray = myArray.concat(myOtherArray);	//[2, 3, 4, 5, 6, 7]
```

- `.join()`创建一个由数组元素使用分隔符连接而成的字符串。

```js
var myArray = ["hello", "world", "!"];
//默认分隔符为逗号
console.log(myArray.join());	//"hello,world,!"
console.log(myArray.join(" "));	//"hello world !"
console.log(myArray.join("!!"));//"hello!!world!!!"

console.log(myArray.join(""));	//"helloworld!"
```

- `.pop()`移除最后一个元素，与`.push()`刚好相反。

```js
var myArray = [];

myArray.push(0);	//[0]
myArray.push(2);	//[0, 2]
myArray.push(7);	//[0, 2, 7]
myArray.pop();		//[0, 2]
```

- `.reverse()`反转数组元素。

```js
var myArray = ["world", "hello"];
myArray.reverse();	//["hello", "world"]
```

- `.shift`移除数组第一个元素。

```js
var myArray = [];

myArray.push(0);	//[0]
myArray.push(2);	//[0, 2]
myArray.push(7);	//[0, 2, 7]

myArray.shift();	//[2, 7]
```

- `.slice()`获取数组的元素，并生成一个新的数组，其中参数为起始位置。

```js
var myArray = [1, 2, 3, 4, 5, 6, 7, 8];
var newArray = myArray.slice(3);

console.log(myArray);	//[1, 2, 3, 4, 5, 6, 7, 8]
console.log(newArray);	//[4, 5, 6, 7, 8]
```

- `.splice()`移除从`index`开始的`length`个元素，并插入可变参数`values`表示的所有数据，至少有三个参数。

```js
myArray.splice(index, length, values, ...);

var myArray = [0, 7, 8, 5];
//从1个位置开始，移除2个元素，并将[1, 2, 3, 4]插入其中
myArray.splice(1, 2, 1, 2, 3, 4); 

console.log(myArray);	//[0, 1, 2, 3, 4, 5]
```

－ `.sort()`数组排序，使用一个比较函数作为参数，缺省按升序排列。

```js
var myArray = [3, 4, 6, 1];
myArray.sort();	//[1, 3, 4, 6]

//返回值大于0表示a > b。
function descending(a, b) {
	return b - a;
}

myArray.sort(descending);	//[6, 4, 3, 1]
```

- `.unshift`在数组第一个位置插入元素。

```js
var myArray = [];

myArray.unshift(0);	//[0]
myArray.unshift(2);	//[2, 0]
myArray.unshift(7);	//[7, 2, 0]
```

- `.forEach()`遍历数组，使用一个拥有三个参数的函数作为参数。

```js
//elem为数组元素
function printElement(elem) {
	console.log(elem);
}

//index为元素位置
function printElementAndIndex(elem, index) {
	console.log("Index " + index + ": " + elem);
}

//array为数组本身
function negateElement(elem, index, array) {
	array[index] = -elem;
}

myArray = [1, 2, 3, 4, 5];

//依次打印数组中的每个元素
myArray.forEach(printElement);

//打印"Index 0: 1", "Index 1: 2", ...
myArray.forEach(printElementAndIndex);

//数组变为[-1, -2, -3, -4, -5]
myArray.forEach(negateElement);
```

更多关于Array的函数请查看[http://learn.jquery.com/arrays/](http://learn.jquery.com/arrays/)。

####函数
函数是一个包含可以重复执行的代码的对象，它可以有0个或多个参数以及一个可选的返回值。可以使用多种方式创建函数。

```js
//声明一个函数
function foo() {
	//Do something
}

var foo = function() {
	//Do something
};
```

有下面几种使用函数的方式。

a. 直接调用函数。

```js
var greet = function(person, greeting) {
	var text = greeting + ", " + person;
    console.log(text);
};

greet("Rebecca", "Hello");	//"Hello, Rebecca"
```

b. 使用函数的返回值。

```js
var greet = function(person, greeting) {
	var text = greeting + ", " + person;
    return text;
};

console.log(greet("Rebecca", "Hello"));	//"Hello, Rebecca"
```

c. 将函数作为返回值。

```js
var greet = function(person, greeting) {
	var text = greeting + ", " + person;
    return function() {
    	console.log(text);
    };
};

var greeting = greeting("Rebecca", "Hello");
greeting();	//"Hello, Rebecca"
```

d. 立即调用的函数（IIFE），JavaScript的函数在创建后可以马上调用。

```js
(function() {
	var foo = "Hello world";
})();
```

e. 函数作为参数传递。

```js
var myFn = function(fn) {
	var result = fn();
    console.log(result);
};

myFn(function() {
	return "Hello, world!";	//"Hello, world!"
});
```

使用命名函数作为参数。

```js
var myFn = function(fn) {
	var result = fn();
    console.log(result);
};

var myOtherFn = function() {
	return "Hello, world!";
};

myFn(myOtherFn);	//"Hello, world!"
```

####jQuery类型检查
jQuery提供基本的类型检查方法。

```js
var myValue = [1, 2, 3];

//JavaScript类型检查操作符typeof检查基本数据类型
typeof myValue === "string";	//false
typeof myValue === "number";	//false
typeof myValue === "undefined";	//false
typeof myValue === "boolean";	//false

myValue === null;	//false

//jQuery类型检查方法
jQuery.isFunction(myValue);		//false
jQuery.isPlainObject(myValue);	//false
jQuery.isArray(myValue);		//true
```

####操作符
- 基本操作符的使用
```js
//字符串连接
var foo = "hello";
var bar = "world";

console.log(foo + " " + bar);

//数值的乘法和除法
2 * 3;
2 / 3;

//自增自减运算
var i = 1;
console.log( ++i) ;	//2
console.log(i);		//2

var i = 1;
console.log(i++);	//1
console.log(i);		//2
```

- 同一个运算符，作用在数值与字符串类型产生的结果可能不同。

```js
var foo = 1;
var bar = "2";
console.log(foo + bar);	//12, 转换为字符串并连接

var foo = 1;
var bar = "2";
console.log(foo + Number(bar));	//3，数值相加
```

>一元(unary)的`+`运算符同样可以将字符串转换为数值。

```js
console.log(foo + +bar);	//3
```

- 逻辑运算符
逻辑运算符包括与`&&`、或`||`，可以用来连接多个操作数进行逻辑判断。

```js
var foo = 1;
var bar = 0;
var baz = 2;

foo || bar;	//返回1，true
bar || foo;	//返回1，true
foo && bar;	//返回0，false
foo && baz;	//返回2，false
baz && foo;	//返回1，true
```

这两个都是二元运算符，如果第一个操作数就能够确定整个表达式的值，则不会去执行第二个表达式。

```js
//如果foo为假，不会执行doSomething()
foo && doSomething(foo);

//如果baz为真，bar === baz
//否则bar == createBar()
var bar = baz || createBar();
```

- 比较运算符

```js
var foo = 1;
var bar = 0;
var baz = "1";
var bim = 2;

foo == bar;	//false
foo != bar;	//true
foo == baz;	//true，类型不同

foo === baz;	//false
foo !== baz;	//true
foo === parseInt(baz);	//true

foo > bim;	//false
foo > baz;	//true
foo <= baz;	//true
```

####分支语句
- 有些代码只会在特定的情况下执行，JavaScript使用`if`、`else`进行流程控制。

```js
//流程控制
var foo = true;
var bar = false;

if (bar) {
	//不会执行
    console.log("戴维营教育");
}

if (bar) {
	//不会执行
} else {
	if (foo) {
    	//执行
    } else {
    	//当bar和foo都为false的时候执行
    }
}
```

- 真假对象
下面一些对象表示假。

```js
false		//逻辑假
""			//空字符串
NaN			//不是一个数值
null		
undefined	//未定义
0			//数值0
```

而其它对象都为真。

```js
"0"
"any string"
[]	//空数组
{}	//空对象
1	/／非0数值
```

- 三目运算符
再一些表达式或简单的判断中，使用三目运算符比`if-else`效率更高。

```js
var bar = 1;
var foo = bar ? 1 : 0;	//foo == 1;
```

- `switch`语句
`switch`可以用来根据一组值执行不同的方法。

```js
switch(foo) {
	case "bar":
    	alert("the value was bar -- yay!");
        break;
        
    case "baz":
    	alert("boo baz :(");
        break;
        
    default:
    	alert("everything else is just ok");
}
```

不过在JavaScript中，有时使用其它方法代替`switch`语句，使得代码可重用性更好。

```js
var stuffToDo = {
	"bar": function() {
    	alert("the value was bar -- yay!");
    },
    
    "baz": function() {
    	alert("boo baz :(");
    },
    
    "default": function() {
    	alert("everything else is just ok");
    }
}

//检查属性是否存在
if (stuffToDo[foo]) {
	stuffToDo[foo]();
}
else {
	stuffToDo["default"]();
}
```

####循环语句
循环语句可以让一个代码块重复执行特定次数。

```js
for (var i = 0; i < 5; i++) {
	//Logs "try 0", "try 1", ..., "try 4".
    console.log("try " + i);
}
```

- `for`循环由四个部分的语句组成。

```js
for ([初始化语句]; [条件判断语句]; [迭代器]) {
	[循环体]
}
```

初始化语句只会执行一次，我们可以在里面为变量设置合适的初始值。而条件判断语句在循环体之前执行。

- `while`循环语句。

```js
while( [判断条件] ) {
	[循环体]
}
```

下面是`while`的一个典型用法。

```js
var i = 0;
while (i < 100) {
	console.log("Currently at " + i);
    i++;
}
```

- `do-while`循环
与`while`循环类似，但是至少会执行一次循环体。

```js
do {
	[循环体]
} while([判断条件]);
```

- `break`与`continue`
一个循环必须等循环条件不再满足才会结束。`break`能够直接结束循环。

```js
for (var i = 0; i < 10; i++) {
	if (something) {
    	break;	//结束循环
    }
}
```

`continue`停止执行当前这次循环中循环体的代码。

```js
for (var i = 0; i < 10; i++) {
	if (something) {
    	continue;	//直接执行i++
    }
    
    console.log("I have been reached");
}
```
