---
title: call和apply
tags: ['JavaScript']
categories: 'JavaScript设计模式和开发实践'
---
## call和apply的区别
call和apply的作用是一模一样的，区别在于传入参数的形式不同。

#### apply
apply接受两个参数，第一个指明函数体内this的指向，第二个参数是一个带下标的集合，可以是数组或者类数组。

```
var func = function( a, b, c ){
    alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};
var arr = [1, 2, 3]
func.apply( null, arr );
```
arr作为参数一起传到func函数并输出。

#### call
call的传入的参数数量时不固定的，但是可以确定的是，第一个参数和apply一样，是函数体内的this指向。后面的参数会被依次传入函数。

```
var func = function( a, b, c ){
    alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};
func.call( null, 1, 2, 3 );
```
我们可以看到，这两种方式最后的结果是一模一样的。但是为什么还要有这两种方法呢？这里就看我们是否知道要传入的参数的数量了，如果不知道，apply是更好地选择，apply的使用率也是更高的，但是如果我们知道，要传入的参数的数量，这时候用call就会更直观，可以直接一目了然的表达出形参和实参的对应关系。

其实call就是一颗包装在apply上面的语法糖。

当传入的第一个参数为null的时候，this的指向就是默认的宿主对象，在浏览器中就是全局对象window，但是如果使用严格模式，就是null

```
var func = function( a, b, c ){
    alert ( this === window ); // 输出 true
};
func.apply( null, [ 1, 2, 3 ] );

var func = function( a, b, c ){
    "use strict";
    alert ( this === null ); // 输出 true
}
func.apply( null, [ 1, 2, 3 ] );
```

当然，call和apply的作用不止于改变this的指向，我们还可以利用它来借用其他对象的方法。

```
Math.max.apply( null, [ 1, 2, 5, 3, 4 ] ) // 输出： 5
```
Math.max()方法本来是要传入一组数，比较出这组数中最大的一个，用了apply后，我们就可以直接得到一个数组中最大的数了，当然，我们还有其他的办法实现，例如：

```
Math.max(...[1, 2, 5, 3, 4])
```

## call和apply的用途
大概上我们可以总结的用途有三个，下面我们会一一来介绍这三种用途
#### 改变this的指向
这是我们最先知道的用途，但是真的理解会用了吗？我们先来看个例子：

```
var obj1 = {
    name: 'sven'
};
var obj2 = {
    name: 'anne'
};
window.name = 'window';
var getName = function(){
    alert ( this.name );
};
getName(); // 输出: window
getName.call( obj1 ); // 输出: sven
getName.call( obj2 ); // 输出: anne
```
这个例子很简单，三次调用分别使用了三个不同的this，清晰明了。
在上一章的this中，我们讨论过一个关于this指向改变的问题，就是document.getElementById，当我们通过这个方法获取到这个对象，并触发这个dom的点击事件时，事件内部的this指向是dom本身

```
document.getElementById( 'div1' ).onclick = function(){
    alert( this.id ); // 输出： div1
};
```
但是，如果事件内部，在定义一个函数，像这样：

```
document.getElementById( 'div1' ).onclick = function(){
    alert( this.id ); // 输出： div1
    var func = function(){
        alert ( this.id ); // 输出： undefined
    }
    func();
};
```
内部的func函数的this不再是dom节点了，变成了全局对象window，这不是我们想要的结果，怎么来改变这个问题呢？这里就可以用call或者apply了

```
document.getElementById( 'div1' ).onclick = function(){
    var func = function(){
        alert ( this.id ); // 输出： div1
    }
    func.call( this );
};
```
再来看看上一节中，我们是怎么处理的：

```
document.getElementById = (function( func ){
    return function(){
    return func.apply( document, arguments );
    }
})( document.getElementById );

var getId = document.getElementById;
var div = getId( 'div1' );
alert ( div.id ); // 输出： div1
```

#### Function.prototype.bind
现代大部分的浏览器都支持Function.prototype.bind，这个函数的作用是创建一个新函数，并指定这个函数的this指向。下面是mdn中对该函数的描述：
> bind() 函数会创建一个新函数（称为绑定函数），新函数与被调函数（绑定函数的目标函数）具有相同的函数体（在 ECMAScript 5 规范中内置的call属性）。当新函数被调用时 this 值绑定到 bind() 的第一个参数，该参数不能被重写。绑定函数被调用时，bind() 也接受预设的参数提供给原函数。一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

下面我们来看一下这个函数是怎么实现的：

```
Function.prototype.bind = function( context ){
    var self = this; // 保存原函数
    return function(){ // 返回一个新的函数
    return self.apply( context, arguments ); // 执行新的函数的时候，会把之前传入的 context
                                            // 当作新函数体内的 this
    }
};
var obj = {
    name: 'sven'
};
var func = function(){
    alert ( this.name ); // 输出： sven
}.bind( obj);
func()
```
这是一个简化版的bind实现，真实的bind肯定要比这个复杂的

#### 借用其他对象的方法
借用方法也是call和apply应用很广泛的用途，下面我们来看第一种场景’借用构造函数‘

```
var A = function( name ){
    this.name = name;
};
var B = function(){
    A.apply( this, arguments );
};
B.prototype.getName = function(){
    return this.name;
};
var b = new B( 'sven' );
console.log( b.getName() ); // 输出： 'sven'
```
这是一个很简单的实现，B借用了A的构造函数，这就相当于实现了一些类似继承的效果。

第二种场景在平常编码中就用到很多次了，就是类数组去借用数组的方法。

```
(function(){
    Array.prototype.push.call( arguments, 3 );
    console.log ( arguments ); // 输出[1,2,3]
})( 1, 2 );
```
arguments是一个类数组元素，通过call方法，我们调用了数组的push方法，为arguments对象插入了一个新的元素，这样的例子很多，我们就不一一去看了。

但是我们需要知道很多对象都可以借用其他对象的方法的，比如例子中的push方法，只要满足两个要求的对象都可以调用这个方法，这两个要求就是：
- 对象本身要可以存取属性；
- 对象的 length 属性可读写。

通过这两个要求我们可以知道，number类型是无法借用push方法的，因为number没有存取属性，还有就是函数不可以，因为函数的length是只读的。


[GitHub博客地址](https://flyliu0723.github.io/fly.blog.io/)






