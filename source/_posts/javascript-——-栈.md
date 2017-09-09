---
title: JavaScript数据结构与算法 —— 栈
date: 2017-09-09 15:04:20
tags:
    - JavaScript
    - 数据结构与算法
categories: Notes
photos:
---
最近看了一本《学习JavaScript数据结构与算法》，想把里面介绍的一些数据结构和算法记录下来，加深印象。

>栈是一种遵从后进先出（LIFO）原则的有序集合。新添加的或待删除的元素都保存在栈的末尾。称作栈顶，另一端就叫栈底。在栈里，新元素都靠近栈顶，就元素都接近栈底。

在JavaScript里面数组的一些方法可以很好的模拟栈。
```javascript
/**
 *用JavaScript数组模拟栈
 *@class Stack
 *@constructor
 *@method
 ** push(element) 添加一个或多个新元素到栈顶
 ** pop() 移除栈顶的元素，同时返回被移除的新元素
 ** peek() 返回栈顶元素，不对栈做任何修改
 ** isEmpty() 返回ture(栈为空) 和 false(非空)
 ** clear() 移除栈里的所有元素
 ** size() 返回栈里元素的个数
 ** print() 输出栈
**/
function Stack() {
  var items = []; //保存栈里的元素
  this.push = function(element) {
    items.push(element);
  }
  this.pop = function() {
    return items.pop();
  }
  this.peek = function() {
    return items[items.length - 1];
  }
  this.isEmpty = function() {
    return !items.length;
  }
  this.clear = function() {
    items = [];
  }
  this.size = function() {
    return items.length;
  }
  this.print = function() {
    console.log(items.toString());
  }
}
```
以上就是一个栈的实现。现在我们来看看怎么使用它。
```javascript
  var stack = new Stack();
  //往栈里添加些元素
  stack.push(3);
  stack.push(4);
  //调用方法
  console.log(stack.isEmpty()); // => false
  console.log(stack.size()); // => 2
  console.log(stack.peek()); // => 4
  console.log(stack.print()); // => '3,4'
  
  stack.pop();
  console.log(stack.print()); // => 3
  
  stack.clear()
  console.log(stack.isEmpty()); // => true
  console.log(stack.print()); // => ''
```
###### 实际应用
进制转换
```javascript
function baseConverter(num, base) {
  var stack = new Stack(),
      base = base || 2,
      rem,
      digits = '0123456789ABCDEF',
      baseString = '';
  while (num > 0) {
    rem = Math.floor(num % base);
    stack.push(rem);
    num = Math.floor(num / base);
  }
  while (!stack.isEmpty()) {
    baseString += digits[stack.pop()];
  }
  return baseString;
}
console.log(baseConverter(100)); //=> 1100100
console.log(baseConverter(100, 8)); //=> 144
console.log(baseConverter(100, 16)); //=> 64
```
