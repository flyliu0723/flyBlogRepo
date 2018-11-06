---
title: 提升
tags: ['JavaScript']
categories: '你不知道的JavaScript'
---

对于一个前端工程师，变量的提升应该是人尽皆知了，看到下面两段代码，肯定很容易得出答案：

```javascript
a = 2;
var a;
console.log(a);
```

```javascript
console.log(a);
var a = 2; 
```

很显然，第一个输出2；第二个输出undefined；当问到为什么会这样时，会毫不犹豫的回答，提升。如果让在详细的说明，也许就说不出个所以然了。

下面我们就好好看看为什么会这样呢？

### 编译器

引擎在解释JavaScript代码之前会首先对其进行编译，而在编译阶段的一部分工作就是找到所有的声明，并用合适的作用域将他们关联起来。

再来看上面的代码，在进行代码的执行之前，‘var a’会被提前处理，也就是首先定义了一个名字为a的变量。所以第一个就是赋值然后输出，所以输出2；而第二个是输出然后赋值，所以是undefined。

> 变量和函数在内的所有声明会在代码被执行前首先被处理。

> > 只有声明本身会被提升，赋值和其他运行逻辑会留在原地。

```javascript
a();
b();
var a = function() {
    console.log('this is a');
}
function b() {
    console.log('this is b');
}
```

既然函数的声明也会提前，我们当然也有看一下声明函数的例子。上面的代码会输出吗？会输出什么？

最后我们得到的记过是

```javascript
a() //Uncaught TypeError: a is not a function
b() // this is b
```

我们来看一下这段代码，由我们知道的知识可以知道a，b的声明都会提前，但是不一样的是a作为一个变量而b则是一个函数，所以a()的输出很容易理解，因为此时的a只是一个变量，所以抛出TypeError。

而b()能执行是因为，函数的声明提升的不止声明，连同提升的还有函数体。

```javascript
c() //Uncaught TypeError: a is not a function
d() // ReferenceError： d is not a function
var c = function d() {
    console.log('this is c and d')
}
```

> ReferenceError代表当一个不存在的变量被引用时发生的错误

再看一下上面这段代码，和上一段有一些微小的差别。c()的错误我们已经知道原因。但是d()又是因为什么呢？

这段代码的提升的只有var c，d的提升并不是我们想象的那样：

```javascript
var c
c()
d()
c = function() {
    var d = ...self...
    console.log('this id c and d')
}
```

### 函数优先

上面我们已经知道了，函数声明和变量声明都会被提前。但是如果发生了函数和变量重名，这时候的会发生什么呢？

```javascript
foo()
var foo
function foo() {
    console.log('this is function')
}
foo = function() {
    console.log('this is variable')
}
```

这时候foo()输出的是‘this is function’ 不是'this is variable'。为什么呢？

答案很简单，函数优先。

var foo 虽然出现在function foo的前面，但是它是重复的声明，会被忽略。

### 结

了解了提升之后，我们需要一个能力：

看到的是：

```javascript
var a = 2	
```

心里想的是：

```javascript
var a
a = 2
```

> **声明本身会提升，而包括函数表达式的赋值在内的赋值操作并不会提升**。

最后总结一下提升：

所有的声明（变量和函数）都会被移动到各自作用域的最顶端。



