---
title: 面向对象的JavaScript
tags: ['JavaScript']
categories: 'JavaScript设计模式和开发实践'
---
## 动态语言类型和鸭子类型

> 从前在 JavaScript 王国里，有一个国王，他觉得世界上最美妙的声音就是鸭子的叫
声，于是国王召集大臣，要组建一个 1000 只鸭子组成的合唱团。大臣们找遍了全国，
终于找到 999 只鸭子，但是始终还差一只，最后大臣发现有一只非常特别的鸡，它的叫
声跟鸭子一模一样，于是这只鸡就成为了合唱团的最后一员。

鸭子类型很好的告诉我们一个原则，在编程过程中，我们不需要知道对象的类型，只需要保证它有某一个方法，调用的时候，它可以执行这个方法就可以了
> 我们只需要关注对象的行为，而不关注对象本身

用代码表示鸭子模型就是：
```
var duck = {
    duckSinging: function(){
    console.log( '嘎嘎嘎' );
    }
};
var chicken = {
    duckSinging: function(){
    console.log( '嘎嘎嘎' );
    }
};
var choir = []; // 合唱团
var joinChoir = function( animal ){
    if ( animal && typeof animal.duckSinging === 'function' ){
        choir.push( animal );
        console.log( '恭喜加入合唱团' );
        console.log( '合唱团已有成员数量:' + choir.length );
    }
};
joinChoir( duck ); // 恭喜加入合唱团
joinChoir( chicken ); // 恭喜加入合唱团
```

## 多态

堕胎的含义是：**同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结
果。** 换句话说就是，当我们给不同的对象发送了同一个消息时，这些对象会根据这个消息做出不同的反馈。

就像我们让一群鸭子和鸡叫的时候，鸭子就会发出‘嘎嘎嘎’的叫声，而鸡就会发出‘咯咯咯’的叫声，同样的‘叫’的指令，得到了不同的声音。


```
var makeSound = function( animal ){
    if ( animal instanceof Duck ){
        console.log( '嘎嘎嘎' );
    }else if ( animal instanceof Chicken ){
        console.log( '咯咯咯' );
    }
};
var Duck = function(){};
var Chicken = function(){};
makeSound( new Duck() ); // 嘎嘎嘎
makeSound( new Chicken() ); // 咯咯咯
```
上面的代码就实现了鸡鸭对叫的指令发出不通反馈的效果，但是这段代码有个问题，就是当我们有了一只狗，狗也要发出声音的时候，我们需要去更改这些代码，这就可能会产生很多影响。
我们需要把‘做什么’和‘谁去做以及怎么去做’分离开来，下面是改进过的代码：
```
var Duck = function(){}
Duck.prototype.sound = function(){
    console.log( '嘎嘎嘎' );
};
var Chicken = function(){}
Chicken.prototype.sound = function(){
    console.log( '咯咯咯' );
};
makeSound( new Duck() ); // 嘎嘎嘎
makeSound( new Chicken() ); // 咯咯咯
```
这样之后，如果来了一条狗，那么我们就可以再加上一段代码，而不需要修改已有的代码：
```
var Dog = function(){}
Dog.prototype.sound = function(){
    console.log( '汪汪汪' );
};
makeSound( new Dog() ); // 汪汪汪
```

这是利用JavaScript实现的多态效果，因为JavaScript是动态语言类型，我们不需要关注对象的类型是什么，但是如果是一个静态的语言类型的语言，比如java呢，我们又要怎么实现多态呢？

静态语言类型设计了一个实现方式---“向上转型”，不管使狗还是鸡鸭，它们都是动物，所以我们可以定义一个超类--‘animal’，这个超类有一个方法叫做‘makesound’，狗还有鸡鸭都继承这个方法，并重写这个方法，这样也就实现了多态。

Java的实现多态我们暂不讨论，接下来我们继续来看JavaScript的多态。

观察上面JavaScript实现的代码我们很容易可以看出，本来我们需要用到if...else的判断语句来判断是鸡还是鸭，优化后的代码就可以消除这些条件判断语句。

换句话说，多态的目的就是**通过把过程化的条件分支语句转化为对象的多态性，从而消除这些条件分支语句**

## 封装

封装的目的是将信息隐藏，封装不仅仅是隐藏数据，还包括隐藏实现细节，设计细节以及隐藏对象的类型等。

## 原型模式和基于原型继承的JavaScript对象系统

原型模式是用于创建对象的一种设计模式，通常我们想要创建一个对象，一种方法时先指定它的类型，然后通过类来创建这个对象。
原型模式则选用了另一个方式，我们不在关心对象的具体类型，而是找到一个对象，然后克隆出来一个一模一样的对象。

可以说JavaScript中所有的对象都是通过克隆出来的，就想人类，最初只是一个受精卵，通过不断地分类克隆，产生了多种多样的细胞，组织，而对于JavaScript来说，最初的那个受精卵就是Object对象。

克隆的这个过程就出现了另外一些概念，这里我们只会提一下，不会深入的讲解

animal对象通过object对象克隆而来，object就是animal的原型，dog对象就克隆自animal，所以animal就是dog的原型，这个时候我们不难发现通过原型，我们链接成了一个链，这条链被称为原型链。

当我们尝试调用dog的某个方法时，如果在dog对象上没有找到这个方法，就会沿着原型链向上查找，这样一来就能得到继承的效果。

原型编程泛型的基本规则
- 所有的数据都是对象
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
- 对象会记住它的原型
- 如果对象无法响应某个请求，就会将这个请求委托给自己的原型



