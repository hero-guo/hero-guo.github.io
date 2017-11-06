---
title: 数据去重
date: 2017-11-06
tags: 
  - JavaScript
  - 数据去重
categories: JavaScript
---
#### 只包含原始变量, 不会判断值的类型
<!-- more -->

```javascript
const array = [ 1, 1, 'a', 'a' ];
function uniq(arr) {
 return arr.filter((v, i) => arr.indexOf(v) === i);
}
console.log(uniq(array)); //=> [1, 'a']
```
```javascript
const array = [ 1, 1, 'a', 'a' ];
function uniq(arr) {
 return Array.from(new Set(arr));
//return [...new Set(arr)]; 
}
console.log(uniq(array)); //=> [1, 'a']
```
```javascript
const array = [ 1, 1, 'a', 'a' ];
function uniq(arr) {
  const res = [];
  const len = arr.length;
  for (let i = 0; i < len; i++) {
    if (!~res.indexOf(arr[i])) res.push(arr[i]);
  }
  return res;
}
console.log(uniq(array)); //=> [1, 'a']
```
#### 包含引用类型
```javascript
const array = [{ a: 1 }, { a: 1 }, [ 1, 2 ], [ 1, 2 ]];
function uniq(arr) {
 const hash = {};
 return arr.filter(v => {
   const key = JSON.stringify(v);
   const bool = !!hash[key];
   return bool ? false : hash[key] = true;
 });
}
console.log(uniq(array)); //=> [{ a: 1 }, [ 1, 2 ]]
```

