---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [JavaScript, jQuery, HTML, Web开发]
---
####基础知识
网页由三个部分组成：HTML、CSS和JavaScript。它们分别完成不同的功能，其中HTML描述页面内容、CSS负责内容的展示、JavaScript添加交互功能和动态效果。三者一起组成一个完整的Web页面。

```html
<!DOCTYPE html>
<html>
	<head>
    	<meta charset="utf-8">
        <title>长沙戴维营教育jQuery教程 第二课</title>
        
        <!-- CSS -->
        <style>
        	h1 {
            	font-size: 14px;
                color: hotpink;
            }
            
            button {
            	color: red;
            }
        </style>
    </head>
    <body>
    	<h1>jQuery教程 第二课</h1>
        <button>点击这里</button>
        
        <!-- Javascript -->
        <script type="text/javascript">
        	var button = document.querySelector("button");
            button.addEventListener("click", function(event){
            	alert("jQuery教程 第二课");
            }, false);
        </script>
    </body>
</html>
```

####开发工具
这三个部分都不需要编译器的支持，因此也不需要特别的开发工具。我们使用文本编辑器、浏览器就可以进行开发和调试了。当然，市面上也有许多集成开发环境，甚至可以进行可视化编辑。推荐使用下面的组合进行开发：

- 编辑器：Sublime Text 2/3、Vim等。
- 浏览器：Firefox、Chrome、Safari、IE等。
- 调试工具：浏览器自带的调试工具。

####编写JavaScript代码
我们可以通过三种方式往网页中添加JavaScript代码：

- 引入外部代码。我们可以将JavaScript代码写在单独的文件中，然后通过`<script>`标签将它包含到一个网页中。这样可以使得代码结构更加清楚，并且能够提高代码的可重用性。JavaScript文件允许被浏览器缓存在本地，因此可以减少页面的加载时间。

```html
<!-- JavaScript文件的扩展名为”.js“ -->
<script src="/path/to/example.js"></script>
```

- 内嵌JavaScript代码。通过`<script>`标签可以直接在网页中编写JavaScript代码。

```html
<!-- 内嵌JavaScript代码 -->
<script>
	alert("戴维营教育jQuery教程 第二课");
</script>
```

- 最后一种方式是将JavaScript代码作为HTML元素的事件处理属性，但是不建议这么使用。

```html
<!-- 事件处理属性 -->
<a href="javascript:alert('戴维营教育jQuery教程');">点击这里!</a>
<button onClick="alert('第二课');">再点这里!</button>
```

####位置与功能
我们可以将JavaScript代码放在任意位置，但是如果它操作了HTML元素的话，需要保证代码执行的时候这些元素已经在页面上存在了。下面是一个常见的错误：

```html
<!DOCTYPE html>
<html>
	<head>
    	<script type="text/javascript">
        	//在元素加载前就进行访问，会导致异常
        	var title = document.getElementById("title");
            console.log(title);
        </script>
    </head>
    <body>
    	<h1 id="title">戴维营教育jQuery教程 第二课</h1>
    </body>
</html>
```

由于网页一般是从上到下进行加载和执行的，上面的Javascript代码会在`<body>`加载之前执行。我们只要将JavaScript代码移动到`h1`之后就可以修复了。

```html
<!DOCTYPE html>
<html>
	<head>
   	</head>
    <body>
    	<h1 id="title">戴维营教育jQuery教程 第二课</h1>
        <script type="text/javascript">
        	var title = document.getElementById("title");
            console.log(title);
        </script>
    </body>
</html>
```