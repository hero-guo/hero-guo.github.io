---
title: javascript 事件详解
date: 2017-02-13 11:17:33
tags:
categories:
---
### 事件冒泡与捕获

1. 冒泡型事件：事件按照从最特定的事件目标到最不特定的事件目标(document对象)的顺序触发。
  IE 5.5: div -> body -> document
  IE 6.0: div -> body -> html -> document
  Mozilla 1.0: div -> body -> html -> document -> window
  
2. 捕获型事件(event capturing)：事件从最不精确的对象(document 对象)开始触发，然后到最精确(也可以在窗口级别捕获事件，不过必须由开发人员特别指定)。
3. DOM事件流：同时支持两种事件模型：捕获型事件和冒泡型事件，但是，捕获型事件先发生。两种事件流会触及DOM中的所有对象，从document对象开始，也在document对象结束。
  DOM事件模型最独特的性质是，文本节点也触发事件(在IE中不会)。
<!-- more -->
    <div>
      <p></p>
    </div>
这两个元素都绑定了click事件，如果用户点击了p，它在div和p上都触发了click事件，那这两个事件处理程序哪个先执行呢？事件顺序是什么？
##### 事件捕获
当你使用事件捕获时，父级元素先触发，子级元素后触发，即div先触发，p后触发。
##### 事件冒泡
当你使用事件冒泡时，子级元素先触发，父级元素后触发，即p先触发，div后触发。
##### W3C模型
W3C模型是将两者进行中和，在W3C模型中，任何事件发生时，先从顶层开始进行事件捕获，直到事件触发到达了事件源元素。然后，再从事件源往上进行事件冒泡，直到到达document。

程序员可以自己选择绑定事件时采用事件捕获还是事件冒泡，方法就是绑定事件时通过addEventListener函数，它有三个参数，type, listener[, options]。第一个参数表示监听事件类型的字符串。第二个参数当所监听的事件类型触发时，会接收到一个事件通知（实现了 Event 接口的对象）对象。listener 必须是一个实现了 EventListener 接口的对象，或者是一个函数。第三个参数若是true，则表示采用事件捕获，若是false，则表示采用事件冒泡
ele.addEventListener('click',doSomething,true)
true=捕获
false=冒泡

##### IE
IE只支持事件冒泡，不支持事件捕获，它也不支持addEventListener函数，不会用第三个参数来表示是冒泡还是捕获，它提供了另一个函数attachEvent。
    
    ele.attachEvent("onclick", doSomething);
    
事件冒泡（的过程）：事件从发生的目标（event.srcElement||event.target）开始，沿着文档逐层向上冒泡，到document为止。

至此我们可以封装一下事件绑定的方法：

    //事件绑定
    function addEvent(target, type, func) {
      if (target.addEventListener) {
        target.addEventListener(type, func, false);
      } else if (target.attachEvent) {
        target.attachEvent('on' + type, func);
      } else {
        target['on' + type] = func;
      }
    }
    //事件移除
    function removeEvent(target, type, func) {
      if (target.removeEventListener) {
        target.removeEventListener(type, func, false);
      } else if (target.detachEvent) {
        target.detachEvent('on' + type, func);
      } else {
        target['on' + type] = null;
      }
    }
    
    

### 事件的传播是可以阻止的：
阻止事件冒泡
    
     if (event.stopPropagation) {
       event.stopPropagation();
     } else {
       event.cancelBubble = true; // 兼容IE
     }
阻止事件的默认行为

    if (event.preventDefault) {
      event.preventDefault(); 
    } else {
      event.returnValue = false; // 兼容IE
    }

### 事件代理
事件委托就是利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。有点就是可以提高性能。
    
    <ul id='ul'>
      <li>1</li>
      <li>2</li>
      <li>3</li>
      <li>4</li>
    </ul>    
    
    <script>
        var oUl = document.getElementById('ul');
        addEvent(oUl, 'click', function(ev) {
          var ev = ev || window.event;
          var target = ev.target || ev.srcElement;
          if (targe.nodeName.toLowerCase() === 'li') {
            console.log(target.innerHTML);
          }
        });
    </script>
### JS观察者模式
这是一种创建松散耦合代码的技术。它定义对象间 一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。由主体和观察者组成，主体负责发布事件，同时观察者通过订阅这些事件来观察该主体。主体并不知道观察者的任何事情，观察者知道主体并能注册事件的回调函数。
    
    var myEvent = (function() {
      var handlers = {};
    
      function on(evt, func) {
        handlers[evt] = handlers[evt] || [];
        handlers[evt].push(func);
      }
      function once(evt, func) {
        handlers[evt] = [];
        handlers[evt].push(func);
      }
      function off(evt, func) {
        var handler = handlers[evt];
        if (handler) {
          for (var i = 0; i < handler.length; i++) {
            if (handler[i] === func) {
              handler.splice(i, 1);
              return;
            }
          }
        }
      }
      function emit(evt, arg) {
        if (handlers[evt]) {
          for (var i = 0; i < handlers[evt].length; i++) {
            handlers[evt][i](arg);
          }
        }
      }
      return {
        on: on,
        once: once,
        off: off,
        emit: emit,
      };
    })();
