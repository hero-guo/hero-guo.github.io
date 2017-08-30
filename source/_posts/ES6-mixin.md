---
title: ES6-MIXIN
date: 2016-12-11 14:57:15
tags: 
  - JavaScript
  - ES6
categories: JavaScript
---

```javascript
	    //通过class实现混入
    //extends 可以继承动态构造的类
    const Mixin = Sup => class extends Sup {
        //构造方法
        //this关键字则代表实例对象
        constructor(...args) {
            //（1）作为函数调用时（即super(...args)），super代表父类的构造函数。
            //（2）作为对象调用时（即super.prop或super.method()），super代表父类。注意，此时super即可以引用父类实例的属性和方法，也可以引用父类的静态方法。
            super(...args);
        }
    }

    class Widget {
        constructor(width, height, $elem) {
            Object.assign(this, {
                width,
                height,
                $elem: null
            });
        }
        render($where) {
             if (this.$elem) {
                 this.$elem.css({
                 width: this.width + 'px',
                 height: this.height + 'px'
                }).appendTo($where);
              }
        }
    }

    class Button extends Mixin(Widget) {
      constructor(width, height, label){
        super(width, height);
        this.label = label;
        this.$elem = $('<button>').text(this.label);
      }
      render($where) {
        super.render($where);
        this.$elem.click(this.onClick);
      }
      onClick() {
        console.log('Button' + this.label + 'clicked!');
      }
    }
    const $body = $(document.body) || document.body;
    let btn = new Button(125, 30, 'hello world');
    btn.render($body);
```
