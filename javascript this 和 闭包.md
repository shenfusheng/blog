
# this的定义
> this是Javascript语言的一个关键字。它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。**this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象**

# 1、纯粹的函数调用

****
~~~javascript
var a = 1;
function test(){
	var b = 2;
	console.log(this.b); //undefined
	console.log(this.a); //1
}
test();
~~~
this指向全局对象，一般情况下指向window，ES5在严格模式下返回undefined

# 2、作为对象的方法调用
----

```javascript
var a = 1;
var o = {
	a : 2,
	test : function(){
		console.log(this.a); //2
	}
}
o.test();
```
```javascript
var a = 1;
var o = {
	a : 2,
	b : {
	test : function(){
		console.log(this.a); //undefined
		}
	}
}
o.b.test();
```
**this不会沿着作用域链向上查找**

```javascript
var a = 1;
var o = {
	a : 2,
	b : {
	a : 3,
	test : function(){
		console.log(this.a);
		}
	}
}
var f = o.b.test;
f(); //1
```
**this永远指向被调用的那个对象** 

# 3、作为构造函数调用

```javascript
function Test(){
	this.a = 1;
}
var o = new Test();
console.log(o.a); //1
```

 **所谓构造函数，就是通过这个函数生成一个新对象（object）。这时，this就指这个新对象。**

## 当构造函数遇到return

```javascript
function Test(){
	this.a = 1;
	return {};
}

function Test(){
	this.a = 1;
	return function(){};
}

function Test(){
	this.a = 1;
	return true;
}

function Test(){
	this.a = 1;
	return null;
}
var o = new Test();
console.log(o.a)
```
**如果返回值是一个对象(除了null和undefined)，那么this指向的就是那个返回的对象，如果返回值不是一个对象那么this还是指向函数的实例。**

```javascript
var name = "The Window";
var object = {
　　name : "My Object",
　　getNameFunc : function(){
　　　　return function(){
　　　　　　return this.name;
　　　　};
　　}
};
alert(object.getNameFunc()());
```
**当作为普通函数调用时，this指向全局**  

# 使用apply，call和bind时的调用

**改变this的指向**

## call 和 apply
```javascript
var o = {
	a : 2,
	b : {
	test : function(x,y){
		console.log(this.a);
		console.log(x+y)
		}
	}
}
var f = o.b.test;
f(3,5); //undefined, 8
```
```javascript
...
f.call(null,3,5); //undefined, 8
```
```javascript
...
f.call(o,3,5); //2, 8
```
```javascript
...
f.apply(o,[3,5]); //2, 8
```

## bind改变this
```javascript
var o = {
	a : 2,
	b : {
	test : function(x,y){
		console.log(this.a);
		console.log(x+y)
		}
	}
}
var f = o.b.test;
f(3,5); //underfined, 8
```
```javascript
...
f.bind(o)(3,5); //2, 8
```
```javascript
...
var m = f.bind(o);
m(3,5); //2, 8
```

# call,apply和bind的区别
* call和apply都是改变上下文中的this并**立即执行**这个函数，但是apply必须以**数组**的形式将参数传入，call则不用。
* bind方法则返回一个**函数**，不会立即执行，并且可以将参数在执行的时候添加。

# call 和apply的经典用法
## 将伪数组转为数组
```javascript
function test(a,b,c){
	var arr = [].slice.call(arguments);
	var newArr = arr.map(function(val){
		return val + 1;
	})
	return newArr;
}
```
## 求数组的最大值或者最小值
```javascript
function arrMax(arr){
	return Math.max.apply(null,arr);
}
function arrMin(arr){
	return Math.min.apply(null,arr);
}
```

## 实现对象的继承
```javascript
function Animal(name) {
  this.name = name;
  this.showName = function () {
    console.log(this.name);
  }
  }
function Cat(name) {
  Animal.call(this, name); 
  }
var cat = new Cat('Black Cat');
cat.showName(); 
```
## 验证是否是数组
```javascript
functionisArray(obj){ 
    return Object.prototype.toString.call(obj) === '[object Array]' ;
} 
```

# bind的经典用法
## 改变this指向
```javascript
var obj = {
	age: 18
}
document.addEventListener('click',function(){
	alert(this.age)
}.bind(obj))
```
## 保存this变量
```javascript
var obj = {
	age: 18,
	sayAge : function(){
		setTimeout(function(){
			alert(this.age)
		}.bind(this),1000)
	}
}
```

# js闭包

> 当一个函数即便在离开了它的词法作用域(Lexical Scope)的情况下，仍然存储并可以存取它的词法作用域(Lexical Scope)，这个函数就构成了闭包。

***
在 js 语言中，闭包的定义可以简化为嵌套定义在函数内部的函数。 

# 普通函数调用 

```javascript
function f1(){
	var a = 0;
	a++;
	console.log(a)
}
var result1 = f1();
var result2 = f1();
```
***
函数中的局部变量仅在函数的执行期间可用。一旦 f1() 执行过后，我们会很合理的认为 a 变量将不再可用，所以都返回 1 

# 全局变量的累加
```javascript
var a = 0;
function f1(){
	a++;
	console.log(a)
}
var result1 = f1(); //1
var result2 = f1(); //2
```



# 用闭包存储变量
```javascript
function f1(){
	var a = 0;
	function f2(){
		a++;
		console.log(a)
	}
	return f2;
}
var result = f1();
result(); //1
result(); //2
```
***
* 两个函数分别返回 1和2，f1中的变量a一直保存在内存中，并没有因为f1执行完毕就被销毁，为什么呢？ 


* 因为f1是f2的父函数，而通过var result = f1();将f2赋值给一个全局变量，以此f2一直在内存中，由于f2依赖于f1，所以f1也就一直没被销毁，保存在内存中。

* 好处：一个变量长期驻扎在内存中，避免全局变量的污染，私有成员的存在


# 用闭包模拟块级作用域
```javascript
function test(num) {
	for (var i = 0; i < num; i++) {
	  //核心代码
	}
	console.log(i) //num 的值
}
```
```javascript
function test(num) {
	//核心代码
	(function(){
	for(var i = 0; i<num; i++) {
	  //核心代码
	}
	})()
	//核心代码结束
	console.log(i) // i is not defined
}
```


## 用闭包模拟私有方法（模块化）
```javascript
var Counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }   
})();

console.log(Counter.value()); /* logs 0 */
Counter.increment();
Counter.increment();
console.log(Counter.value()); /* logs 2 */
Counter.decrement();
console.log(Counter.value()); /* logs 1 */
```

该共享环境创建于一个匿名函数体内，该函数一经定义立刻执行。环境中包含两个私有项：名为 privateCounter 的变量和名为 changeBy 的函数。 这两项都无法在匿名函数外部直接访问。必须通过匿名包装器返回的三个公共函数访问。

# 增强的模块方式
```javascript
var module = (function(){
	var privateValue = 'private';
	var privateFunc = function(){
			alert(privateValue);
		};
	var app = {};
	app.publicFunc = function(){
		alert('pubilc');
	};
	return app;
})();
module.publicFunc();
```

# 使用闭包的注意点
1. 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露

1. 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。
