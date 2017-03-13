---
title: promise.all错误处理
date: 2017-03-13 21:15:15
tags:
categories:
---

通常处理多个请求的时候我们会用Promise.all()方法。该方法指当所有在可迭代参数中的 promises 已完成，或者第一个传递的 promise（指 reject）失败时，返回 promise。但是当其中任何一个被拒绝的话。主Promise.all([..])就会立即被拒绝，并丢弃来自其他所有promis的全部结果。
<!-- more -->

    var p1 = Promise.resolve(3);
    var p2 = Promise.reject(2);
    var p3 = new Promise((resolve, reject) => {
      setTimeout(resolve, 100, "foo");
    }); 
    
    Promise.all([p1, p2, p3]).then(values => { 
      console.log(values); // 永远走不到这里
    }).catch(function(err) {
      console.log(err); // 2
    });
这不是我们想要的。所以在使用这个方法的时候要记住为每个promise关联一个错误的处理函数
    
    var p1 = Promise.resolve(3).catch(function(err) {
      return err;
    });
    var p2 = Promise.reject(2).catch(function(err) {
      return err;
    });
    var p3 = new Promise((resolve, reject) => {
      setTimeout(resolve, 100, "foo");
    }).catch(function(err) {
      return err;
    }); 
    
    Promise.all([p1, p2, p3]).then(values => { 
      console.log(values); // [3, 2, "foo"]
    }).catch(function(err) {
      console.log(1); //不会走到这里
    });
