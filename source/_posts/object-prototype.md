---
title: 类的继承
categories: categories
date: 2017-02-12 14:57:15
---

1. 混合复制

		//复制source对象的属性
		function mixin(source, target) {
			for (var key in source) {
				if (!(key in target)) {
					target[key] = source[key];
				}
			}
			return target;
		}

		var Vehicle = {
			engines: 1,
			ignition: function() {
				console.log(" Turning on my forward! ");
			},
			drive: function() {
				this.ignition();
				console.log(" Steering and moving forward! ");
			}
		};

		//Car复制Vehicle的属性值和函数引用
		var Car = mixin(Vehicle, {
			wheels: 4,
			drive: function() {
			//显示指定Vehicle对象调用dirve方法
			//如果直接执行Vehicle.dirve()，函数调用中的this就会绑定到Vehicle上，所以使用.call(this)来确保dirve在Car对象的上下文中执行;
			Vehicle.drive.call(this);
				console.log(" Rolling on all " + this.wheels + "wheels!");
			}
		});
		Car.drive();
<!-- more -->
2. 寄生继承
	首先复制一份父类（Vehicle），然后混入子类（Car）。然后构建实例。

		//类
		function Vehicle() {
			this.engines = 1;
		}
		Vehicle.prototype.ignition = function() {
			console.log(" Turning on my forward! ");
		};
		Vehicle.prototype.drive = function() {
			this.ignition();
			console.log(" Steering and moving forward! ");
		};

		//寄生类Car
		function Car() {
			//car 是一个Vehicle
			var car = new Vehicle();

			car.wheels = 4;
			//保存Vehicle::drive()的特殊引用
			var vehDrive = car.drive;

			//重写Vehicle::drive()
			car.drive = function() {
				vehDrive.call(this);
				console.log(" Rolling on all " + this.wheels + "wheels!");
			}
			return car;
		}
		// 因为没事new创建的新对象而是使用返回的car，所以可以直接调用Car();
		//var myCar = Car();
		var myCar = new Car();
		myCar.drive();


### 原型继承


	function Foo(name) {
		this.name = name;
	}

	Foo.prototype.myName = function() {
		return this.name;
	}

	function Bar(name, label) {
		Foo.call(this, name);
		this.label = label;
	}
	//Bar.prototype 关联到 Foo.prototype
	//调用Object.create()创建一个新对象并把内部的[[prototype]]关联到指定的对象（Foo.prototype）
	Bar.prototype = Object.create(Foo.prototype);
	//ES6直接修改现有的Bar.prototype
	//Object.setPrototypeOf(Bar.prototype, Foo.prototype);
	Bar.prototype.myLabel = function() {
		return this.label;
	}

	var a = new Bar('a', 'obj a');

`Bar.prototype = Foo.prototype` 并不会创建一个关联到Bar.prototype的新对象，只是直接引用Foo.prototype对象。因此执行复制语句可能会修改Foo.prototype本身。
`Bar.prototype = new Foo()`虽然会创建一个关联到Bar.prototype的新对象，但是他使用的是new Foo()构造函数调用，如果Foo函数有一些副作用可能会影响到Bar()的后代。
