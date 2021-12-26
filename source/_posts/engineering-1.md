---
title: 前端现代模块化机制
top: true
date: 2021-12-27 01:50:21
tags: Engineering
---

# 前端现代模块化机制

---

模块化的好处：

1. 避免命名冲突
2. 代码结构分离，高可维护性、高可复用性
3. 按需加载

## 前端模块化的演变

---

function 模式 -> 命名空间（namespace） -> 自执行函数（IIFE） -> IIFE 增强 -> CommonJS、AMD、CMD、ES6 模块化

### 1. 全局的 function 模式

```javascript
function foo1() {
  // do something …
}

function foo2() {
  // do something …
}
```

利用了函数作用域的特性来分割变量和实现不同的功能

**缺点：** 函数都声明在了全局环境下，污染全局变量

### 2. 命名空间（namespace）模式

```javascript
const module = {
  data: 1,
  sayName: function (name) {
    console.log("hello, " + name);
  },
  add: function (num1, num2) {
    return num1 + num2;
  },
};

module.data = 2; // 修改了 module 中的数据
module.sayName("noko");
```

将变量和函数作为一个对象的属性，整合起来的对象就是一个模块，

**缺点：** 数据不安全，容易篡改模块对象中的数据和方法

### 3. 自执行函数（IIFE）和闭包

```javascript
;(function(window){
	let data = 1

	const setData = (newData) => {
		data = newData
	}

	const getData = () => console.log(data)

	window.module = {
		setData,
		getData
	}
}
})(window)

module.getData() // 1
module.setData(2)
module.getData() // 2
```

数据私有化，外部只能通过暴露的方法获取和修改

**缺点：** 没有引入依赖的机制

### 4. IIFE 增强版 —— 现代模块化的基础

```javascript
;(function(window, $){
	let data = 1

	const setData = (newData) => {
		data = newData
	}

	const getData = () => console.log(data)

	const changeColor = () => {
		$('body').css('background', 'red')
	}

	window.module = {
		setData,
		getData,
		changeColor
	}
}
})(window, jQuery)

window.changeColor()
```

模块互相独立，模块也可以引进另外一个模块，依赖关系清晰

**缺点：** 引入其他模块或者第三方库时，要注意引入的先后顺序

```javascript
// html 文件按顺序引入
<script type="text/javascript" src="jquery-1.10.1.js"></script>
<script type="text/javascript" src="module.js"></script> <script type="text/javascript">
	module.changeColor()
</script>
```

## 前端现代模块化

---

前端目前模块化规范：CommonJS、AMD、CMD、ES6 模块化

### CommonJS

一个文件就是一个模块，模块间数据互相独立。

在服务器端，运行时同步加载模块；浏览器端，模块需编译处理。

四个重要对象：

1. module

每个模块内部都有一个 module 对象

> module.id 模块的识别符，通常是带有绝对路径的模块文件名。 module.filename 模块的文件名，带有绝对路径。module.loaded 返回一个布尔值，表示模块是否已经完成加载。
> module.parent 返回一个对象，表示调用该模块的模块。
> module.children 返回一个数组，表示该模块要用到的其他模块。module.exports 表示模块对外输出的值。

1. exports

exports 是 `module.exports` 的引用，指向 module 对象下的属性 exports，相当于

```javascript
let exports = module.exports;
```

避免以下引起 exports 直接复制，切断了与 module.exports 的联系

```javascript
exports.hello = function () {
  return "hello";
};

module.exports = "Hello world";
```

**所以不建议使用 exports，而建议使用 module.exports 导出模块。**

1. require

> 用于加载 module.exports 的模块

1. global

> 所有模块都能访问到 global 对象

**使用场景：** Node.js、小程序、浏览器（Browserify）

**特点：** 

1. 运行时加载，并且第一次加载模块后会被缓存

```javascript
require("./example.js"); // 程序执行到这里开始加载，并且缓存到 require.cache 对象

require("./example.js").message = "hello";
require("./example.js").message;
```

程序执行到这里开始加载，并且缓存到 `require.cache` 对象，所有的模块缓存都在这个对象中。

1. 同步加载模块

```javascript
let a = require("./a"); // 执行到此处时，a.js 才同步下载并执行
```

在浏览器中可以使用 browserify 实现，由于 CommonJS 同步加载的方式，而同步意味着阻塞加载，所以在浏览器环境并不适用；而服务器文件都在在本地，加载速度快，CommonJS 是适用的。

**举一个栗子 🌰：** 

```javascript
// 定义模块math.js
let basicNum = 0;
function add(a, b) {
  return a + b;
}
module.exports = {
  //在这里写上需要向外暴露的函数、变量
  add: add,
  basicNum: basicNum,
};
```

```javascript
// 引用自定义的模块时，参数包含路径，可省略.js
let math = require('./math');
math.add(2, 5);

// 引用核心模块时，不需要带路径
var http = require('http');
http.createService(...).listen(3000);
```

### AMD 和 require.js

由于 CommonJS 的同步加载的机制并不适用于浏览器，从而出现了 AMD 异步加载的模块化规范。用于浏览器 require.js，是 AMD 规范的实现。

1. 定义暴露模块

```javascript
//定义没有依赖的模块
define(function () {
  return 模块;
});
```

```javascript
//定义有依赖的模块
define(["module1", "module2"], function (m1, m2) {
  return 模块;
});
```

1. 引入使用模块

```javascript
require(["module1", "module2"], function (m1, m2) {
  // 使用m1/m2
});
```

**使用场景：** 浏览器

**特点：** 

1. 异步加载，提前执行依赖
