---
title: this
tags: ['JavaScript']
categories: 'JavaScript设计模式和开发实践'
---
## this的指向
this的指向大致可分为四种：
1. 作为对象的方法调用
2. 作为普通函数调用
3. 构造器调用
4. function.prototype.call或function.prototype.apply调用
下面我们来一个一个的学习

### 作为对象的方法调用
当函数作为对象的方法调用，this指向该对象
```
var obj = {
    a: 1,
    getA: function(){
        alert ( this === obj ); // 输出： true
        alert ( this.a ); // 输出: 1
    }
};
obj.getA();
```

### 作为普通函数调用
这种情况下的this指向全局对象，也就是window对象

```
window.name = 'globalName';
var getName = function(){
    return this.name;
};
console.log( getName() ); // 输出： globalName
```
或者
```
window.name = 'globalName';
var myObject = {
    name: 'sven',
    getName: function(){
        return this.name;
    }
};
var getName = myObject.getName;
console.log( getName() ); // globalName
```

这里需要注意的是，在严格模式下，这种情况的this不会指向全局对象，而是undefined
```
function func(){
    "use strict"
    alert ( this ); // 输出： undefined
}
func();
```

### 构造器调用
```
var myClass = function(){
    this.name = '666'
}
var obj1 = new myClass();
obj1.name //"666"
var obj2 = myClass
obj2.name//"myClass"
obj2
/*ƒ (){
    this.name = '666'
}*/
var obj3 = myClass()
obj3.name
/*VM1898:1 Uncaught TypeError: Cannot read property 'name' of undefined
    at <anonymous>:1:6
(anonymous) @ VM1898:1*/
obj3//undefined
```
上面代码是在控制台敲的，用来查看new关键字的作用

除了宿主提供的一些内置函数，大部分的JavaScript函数都可以当做构造器使用。构造器的外表和普通函数一模一样，区别在于调用的方式。当被new运算符调用函数时，该函数会**返回一个对象**，构造器里的this就指向返回的这个对象
```
var MyClass = function(){
    this.name = 'sven';
};
var obj = new MyClass();
alert ( obj.name ); // 输出： sve
```
这里有一个需要注意的点，就是如果构造器显示的返回一个object类型的对象，那么运算结果就是返回的这个对象，不是我们之前说的this了
```
var MyClass = function(){
    this.name = 'sven';
    return { // 显式地返回一个对象
        name: 'anne'
    }
};
var obj = new MyClass();
alert ( obj.name ); // 输出： anne
```
只有构造器不显式的返回任何数据，或者返回的是一个非对象类型的数据时，才会得到我们想要的this。

### function.prototype.call或function.prototype.apply调用
这两种方式可以动态的改变传入的this
```
var obj1 = {
    name: 'sven',
    getName: function(){
        return this.name;
    }
};
var obj2 = {
    name: 'anne'
};
console.log( obj1.getName() ); // 输出: sven
console.log( obj1.getName.call( obj2 ) ); // 输出： anne
```

## 丢失的this

```
var obj = {
    myName: 'sven',
    getName: function(){
        return this.myName;
    }
};
console.log( obj.getName() ); // 输出： 'sven'
var getName2 = obj.getName;
console.log( getName2() ); // 输出： undefined
```
上面的输出结果我们不会感到意外，因为一个事对象调用，而另一个是直接调用，但是
在document.getElementById等方法上有一个坑，我们很难发现。


```
var result = document.getElementById('result')
result.id  //"result"
```
上面代码获取到了id是result的dom，但是调用的方法太长了，我们可不可以改变一下呢？于是我们写出了这样的代码：

```
var getId = function( id ){
return document.getElementById( id );
};
getId( 'div1' );
```
这是一些框架采用的方法，这么做也是可以的，但是我们又想能不能更简单：
```
var getId = document.getElementById
getId('div')
```
这样呢？可不可以？

在各个浏览器中试了之后我们就会发现出错了。为什么呢？

答案就是， document.getElementById这个方法中用到了this，这时候的this是指向document的，但是我们赋值给了getId之后，调用getId就变成了普通函数的调用，这时候的this就指向了全局对象，所以就出错了。

我们可以重写 document.getElementById这个方法，利用apply把document当做this传入getId函数，这样就可以实现了

```
document.getElementById = (function( func ){
return function(){
return func.apply( document, arguments );
}
})( document.getElementById );
var getId = document.getElementById;
var div = getId( 'div1' );
alert (div.id); // 输出： div1
```





