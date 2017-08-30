---
title: JavaScript 面向对象
date: 2017-04-11 11:05:34
tags: JavaScript
categories: JavaScript
---
### 构造函数
构造函数也是一个函数，通过new运算符可以生成一份实例同时把this绑定到实例上。JavaScript规定，每一个构造函数都有一个prototype属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。
```javascript
  function Vehicle (engines) {
        this.engines = engines;
    }
    Vehicle.prototype.drive = function() {
        console.log(`engines=>${this.engines}`);    
    }
    //实例
    var car1 = new Vehicle(1);
    var car2 = new Vehicle(2);
    car1.drive(); //=> engines=>1
    car2.drive(); //=> engines=>2
    console.log(car1.constructor === Vehicle); //true
    console.log(car2.constructor === Vehicle); //true
```
<!-- more -->
instanceof是一个操作符，可以判断对象是否为某个类型的实例
```javascript
    console.log(car1 instanceof Vehicle); //true
    console.log(car2 instanceof Vehicle); //true
```
instanceof判断的是对象
```javascript
    1 instanceof Number //false
```

isPrototypeOf用来判断proptotype对象和实例之间的关系
```javascript
  console.log(Vehicle.prototype.isPrototypeOf(car1)); //true
    console.log(Vehicle.prototype.isPrototypeOf(car2)); //true
```

hasOwnProperty()可以判断一个对象是否包含自定义属性而不是原型链上的属性
```javascript
   console.log(car1.hasOwnProperty('engines')); //true
    console.log(car1.hasOwnProperty('drive')); //false

```
