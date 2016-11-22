---
title: ECMAScript 2015 学习记录
tags: ECMAScript
---

# ECMAScript 2015 学习记录

1.新增let命令，用来声明变量，同var类似，但是所声明的变量只在命令所在的代码块有效。不存在变量提升现象。在变量声明之前使用会报错。

	{
		let a = 1;
		var b = 1;
	}

	//a is not defined
	//b 1
