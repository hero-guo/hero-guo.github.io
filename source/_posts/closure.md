---
title: 充分理解闭包
date: 2017-02-12 14:57:15
tags: 
  - JavaScript
  - 闭包
categories: Notes
---
### 闭包定义
* 函数定义时的作用域链到函数执行时依然有效
* 函数体内部到变量都可以保存在函数作用域内
* 闭包可以访问它被创建时候到上下文环境

<!-- more -->
### 作用域

由于闭包是基于词法作用域书写代码时所产生到自然结果，所以为了理解闭包，我们先来看看什么是词法作用域。

##### 什么是词法作用域？
大部分标准语言编译器到第一个工作阶段叫做词法化。简单的说词法作用域就是定义在词法阶段到作用域（也就是说函数声明的位置）。


```javascript
function foo(a) {
  var b = a * 2;
  
  function bar(c) {
    console.log(a, b, c);
  }
  
  bar(b * 3);
}
foo(2); //=> 2, 4, 12
```


在上面到代码中有3个逐级嵌套到作用域。

1. 包含着整个全局作用域，其中只有一个标识符： foo；
2. 包含foo所创建到作用域，其中有三个标识符：a、bar、b；
3. 包含bar所创建到作用域，其中有一个标识符：c。

在引擎执行console.log(...)声明，并查找a、b、c三个变量到引用时。首先从最内部到作用域也就是bar()函数作用域开始查找。引擎无法找到a就会继续向上查找也就是foo()函数作用域。在这里找到来a。对于b来讲也是一样的。作用域查找会在找到第一个匹配的标识符时停止。

##### 闭包的产生
当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前的词法作用域之外执行。
```javascript
    function foo() {
      var a = 2;
      function bar() {
        console.log(a);
      }
      return bar;
    }
    var baz = foo();
    baz(); // => 2   这就是闭包
```
函数bar的词法作用域能够访问foo的内部作用域。然后我们返回bar本身。当foo执行后，其返回值（也就是bar函数）赋值给变量baz并调用baz，实际上只是通过不同的标识符引用了内部的函数bar。虽然bar是在定义的词法作用域以外被调用，但是闭包可以访问它被创建时候到上下文环境，所以还是可以访问到变量a。这是为什么呢？

在foo执行之后。通常foo到整个内部作用域都会被销毁，因为引擎有垃圾回收机制用来释放不在使用到内存空间。但是因为bar声明在foo的内部，它拥有涵盖foo内部作用域的闭包，是的该作用域一直存在，并被bar随时引用。

无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。

 ```javascript
    function foo() {
      var a = 2;
      function baz() {
        console.log(a);
      }
      bar(baz);
    }
    function bar(fn) {
      fn(); //闭包
    }
```
##### 随处可见到闭包

你已经写过到代码中一定到处都是闭包到影子。
```javascript
    function wait(message) {
      setTimeout(function timer() {
        console.log(message);
      }, 1000)
    }
    wait('Hello, closure');
```

wait执行1000毫秒后，它到内部作用域并不会消失，timer函数依然可以访问它内部到变量message。
```javascript
    function setupBot(name, selector) {
      $(selector).click(function activator() {
        console.log('Activating:' + name);
      });
    }
    setupBot('Closure Bot 1', '#bot1');
```
本质上无论何时何地。如果将函数当作第一级到值类型并到处传递，你就会看到闭包了。

##### 循环和闭包
```javascript
    for (var i=1; i<=5; i++) {
      setTimeout(function() {
        console.log(i);
      }, i*1000)
    }
```
这段代码期望是输出1～5，每秒一次，每次一个。但是实际上是每秒一次输出到都是6。这是为什么呢？

原来循环中定义的函数都共享一个全局作用域，因此实际上只有一个i。因此我们需要更多的闭包作用域，特别是循环过程中每个迭代都需要一个闭包作用域。
```javascript
    for(var i=1; i<=5; i++) {
      (function(j) {
        setTimeout(function() {
          console.log(j)
        }, j*1000)
      })(i);
    }
```
使用let可以有更简单都写法
```javascript
    for(let i=1; i<=5; i++) {
      setTimeout(function() {
        console.log(i);
      }, i*1000)
    }
```
这是因为let每次迭代都会重新声明一个作用域。
### 参考
《你不知道的javascript上》

《javascript权威指南》

《javascript高级程序设计》

《javascript语言精粹》
