---
title: javascript原型与继承
date: 2019-07-01 20:08:01
tags: [prototype, javascript]
categories: javascript
---
对于使用过基于类的语言 (如 Java 或 C++) 的开发人员来说，JavaScript 有点令人困惑，因为它是动态的，并且本身不提供一个 class 实现。（在 ES2015/ES6 中引入了 class 关键字，但那只是语法糖，JavaScript 仍然是基于原型的）。  

当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（prototype ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。  

几乎所有 JavaScript 中的对象都是位于原型链顶端的 Object 的实例。  

尽管这种原型继承通常被认为是 JavaScript 的弱点之一，但是原型继承模型本身实际上比经典模型更强大。例如，在原型模型的基础上构建经典模型相当简单。  
<!--more-->

---
### 基于原型链的继承
#### 继承属性
JavaScript 对象是动态的属性“包”（指其自己的属性）。JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。  
> 遵循ECMAScript标准，someObject.[[Prototype]] 符号是用于指向 someObject 的原型。从 ECMAScript 6 开始，[[Prototype]] 可以通过 Object.getPrototypeOf() 和 Object.setPrototypeOf() 访问器来访问。这个等同于 JavaScript 的非标准但许多浏览器实现的属性 __proto__。
但它不应该与构造函数 func 的 prototype 属性相混淆。被构造函数创建的实例对象的 [[prototype]] 指向 func 的 prototype 属性。Object.prototype 属性表示 Object 的原型对象。

这里演示当尝试访问属性时会发生什么：
``` javascript
// 让我们从一个自身拥有属性a和b的函数里创建一个对象o：
let f = function() {
   this.a = 1;
   this.b = 2;
}
/* 这么写也一样
function f() {
  this.a = 1;
  this.b = 2;
}
*/
let o = new f(); // {a: 1, b: 2}

// 在f函数的原型上定义属性
f.prototype.b = 3;
f.prototype.c = 4;

// 不要在 f 函数的原型上直接定义 f.prototype = {b:3,c:4};这样会直接打破原型链
// o.[[Prototype]] 有属性 b 和 c
//  (其实就是 o.__proto__ 或者 o.constructor.prototype)
// o.[[Prototype]].[[Prototype]] 是 Object.prototype.
// 最后o.[[Prototype]].[[Prototype]].[[Prototype]]是null
// 这就是原型链的末尾，即 null，
// 根据定义，null 就是没有 [[Prototype]]。

// 综上，整个原型链如下: 

// {a:1, b:2} ---> {b:3, c:4} ---> Object.prototype---> null

console.log(o.a); // 1
// a是o的自身属性吗？是的，该属性的值为 1

console.log(o.b); // 2
// b是o的自身属性吗？是的，该属性的值为 2
// 原型上也有一个'b'属性，但是它不会被访问到。
// 这种情况被称为"属性遮蔽 (property shadowing)"

console.log(o.c); // 4
// c是o的自身属性吗？不是，那看看它的原型上有没有
// c是o.[[Prototype]]的属性吗？是的，该属性的值为 4

console.log(o.d); // undefined
// d 是 o 的自身属性吗？不是，那看看它的原型上有没有
// d 是 o.[[Prototype]] 的属性吗？不是，那看看它的原型上有没有
// o.[[Prototype]].[[Prototype]] 为 null，停止搜索
// 找不到 d 属性，返回 undefined
```

#### 继承方法
JavaScript 并没有其他基于类的语言所定义的“方法”。在 JavaScript 里，任何函数都可以添加到对象上作为对象的属性。函数的继承与其他的属性继承没有差别，包括上面的“属性遮蔽”（这种情况相当于其他语言的方法重写）。  

当继承的函数被调用时，this 指向的是当前继承的对象，而不是继承的函数所在的原型对象。
```
var o = {
  a: 2,
  m: function(){
    return this.a + 1;
  }
};

console.log(o.m()); // 3
// 当调用 o.m 时，'this' 指向了 o.

var p = Object.create(o);
// p是一个继承自 o 的对象

p.a = 4; // 创建 p 的自身属性 'a'
console.log(p.m()); // 5
// 调用 p.m 时，'this' 指向了 p
// 又因为 p 继承了 o 的 m 函数
// 所以，此时的 'this.a' 即 p.a，就是 p 的自身属性 'a'
```
### 在 JavaScript 中使用原型
下去，来仔细分析一下这些应用场景下， JavaScript 在背后做了哪些事情。  

正如之前提到的，在 JavaScript 中，函数（function）是允许拥有属性的。所有的函数会有一个特别的属性 —— prototype 。请注意，以下的代码是独立的（出于严谨，假定页面没有其他的JavaScript代码）。为了最佳的学习体验，我们强烈建议阁下打开浏览器的控制台（在Chrome和火狐浏览器中，按Ctrl+Shift+I即可），进入“console”选项卡，然后把如下的JavaScript代码复制粘贴到窗口中，最后通过按下回车键运行代码。
```
function doSomething(){}
console.log( doSomething.prototype );
// 和声明函数的方式无关，
// JavaScript 中的函数永远有一个默认原型属性。
var doSomething = function(){};
console.log( doSomething.prototype );
```
在控制台显示的JavaScript代码块中，我们可以看到doSomething函数的一个默认属性prototype。而这段代码运行之后，控制台应该显示类似如下的结果：
```
{
    constructor: ƒ doSomething(),
    __proto__: {
        constructor: ƒ Object(),
        hasOwnProperty: ƒ hasOwnProperty(),
        isPrototypeOf: ƒ isPrototypeOf(),
        propertyIsEnumerable: ƒ propertyIsEnumerable(),
        toLocaleString: ƒ toLocaleString(),
        toString: ƒ toString(),
        valueOf: ƒ valueOf()
    }
}
```
我们可以给doSomething函数的原型对象添加新属性，如下：
```
function doSomething(){}
doSomething.prototype.foo = "bar";
console.log( doSomething.prototype );
```
可以看到运行后的结果如下：
```
{
    foo: "bar",
    constructor: ƒ doSomething(),
    __proto__: {
        constructor: ƒ Object(),
        hasOwnProperty: ƒ hasOwnProperty(),
        isPrototypeOf: ƒ isPrototypeOf(),
        propertyIsEnumerable: ƒ propertyIsEnumerable(),
        toLocaleString: ƒ toLocaleString(),
        toString: ƒ toString(),
        valueOf: ƒ valueOf()
    }
}
```
现在我们可以通过new操作符来创建基于这个原型对象的doSomething实例。使用new操作符，只需在调用doSomething函数语句之前添加new。这样，便可以获得这个函数的一个实例对象。一些属性就可以添加到该原型对象中。  

请尝试运行以下代码：
```
function doSomething(){}
doSomething.prototype.foo = "bar"; // add a property onto the prototype
var doSomeInstancing = new doSomething();
doSomeInstancing.prop = "some value"; // add a property onto the object
console.log( doSomeInstancing );
```
运行的结果类似于以下的语句。
```
{
    prop: "some value",
    __proto__: {
        foo: "bar",
        constructor: ƒ doSomething(),
        __proto__: {
            constructor: ƒ Object(),
            hasOwnProperty: ƒ hasOwnProperty(),
            isPrototypeOf: ƒ isPrototypeOf(),
            propertyIsEnumerable: ƒ propertyIsEnumerable(),
            toLocaleString: ƒ toLocaleString(),
            toString: ƒ toString(),
            valueOf: ƒ valueOf()
        }
    }
}
```
如上所示, doSomeInstancing 中的__proto__是 doSomething.prototype. 但这是做什么的呢？当你访问doSomeInstancing 中的一个属性，浏览器首先会查看doSomeInstancing 中是否存在这个属性。  

如果 doSomeInstancing 不包含属性信息, 那么浏览器会在 doSomeInstancing 的 __proto__ 中进行查找(同 doSomething.prototype). 如属性在 doSomeInstancing 的 __proto__ 中查找到，则使用 doSomeInstancing 中 __proto__ 的属性。  

否则，如果 doSomeInstancing 中 __proto__ 不具有该属性，则检查doSomeInstancing 的 __proto__ 的  __proto__ 是否具有该属性。默认情况下，任何函数的原型属性 __proto__ 都是 window.Object.prototype. 因此, 通过doSomeInstancing 的 __proto__ 的  __proto__  ( 同 doSomething.prototype 的 __proto__ (同  Object.prototype)) 来查找要搜索的属性。  

如果属性不存在 doSomeInstancing 的 __proto__ 的  __proto__ 中， 那么就会在doSomeInstancing 的 __proto__ 的  __proto__ 的  __proto__ 中查找。然而, 这里存在个问题：doSomeInstancing 的 __proto__ 的  __proto__ 的  __proto__ 其实不存在。因此，只有这样，在 __proto__ 的整个原型链被查看之后，这里没有更多的 __proto__ ， 浏览器断言该属性不存在，并给出属性值为 undefined 的结论。  

让我们在控制台窗口中输入更多的代码，如下：
```
function doSomething(){}
doSomething.prototype.foo = "bar";
var doSomeInstancing = new doSomething();
doSomeInstancing.prop = "some value";
console.log("doSomeInstancing.prop:      " + doSomeInstancing.prop);
console.log("doSomeInstancing.foo:       " + doSomeInstancing.foo);
console.log("doSomething.prop:           " + doSomething.prop);
console.log("doSomething.foo:            " + doSomething.foo);
console.log("doSomething.prototype.prop: " + doSomething.prototype.prop);
console.log("doSomething.prototype.foo:  " + doSomething.prototype.foo);
```
结果如下：
```
doSomeInstancing.prop:      some value
doSomeInstancing.foo:       bar
doSomething.prop:           undefined
doSomething.foo:            undefined
doSomething.prototype.prop: undefined
doSomething.prototype.foo:  bar
```

#### 性能
在原型链上查找属性比较耗时，对性能有副作用，这在性能要求苛刻的情况下很重要。另外，试图访问不存在的属性时会遍历整个原型链。  

遍历对象的属性时，原型链上的每个可枚举属性都会被枚举出来。要检查对象是否具有自己定义的属性，而不是其原型链上的某个属性，则必须使用所有对象从 Object.prototype 继承的 hasOwnProperty 方法。下面给出一个具体的例子来说明它：
```
console.log(g.hasOwnProperty('vertices'));
// true

console.log(g.hasOwnProperty('nope'));
// false

console.log(g.hasOwnProperty('addVertex'));
// false

console.log(g.__proto__.hasOwnProperty('addVertex'));
// true
```
hasOwnProperty 是 JavaScript 中唯一一个处理属性并且不会遍历原型链的方法。（译者注：原文如此。另一种这样的方法：Object.keys()）  

注意：检查属性是否为 undefined 是不能够检查其是否存在的。该属性可能已存在，但其值恰好被设置成了 undefined。

### 结论
在编写使用复杂代码之前，理解原型继承模型是至关重要的。此外，请注意代码中原型链的长度，并在必要时将其分解，以避免可能的性能问题。此外，原生原型不应该被扩展，除非它是为了与新的 JavaScript 特性兼容。  
### 示例
B 继承自 A：
```
function A(a){
  this.varA = a;
}

// 以上函数 A 的定义中，既然 A.prototype.varA 总是会被 this.varA 遮蔽，
// 那么将 varA 加入到原型（prototype）中的目的是什么？
A.prototype = {
  varA : null,
/*
既然它没有任何作用，干嘛不将 varA 从原型（prototype）去掉 ? 
也许作为一种在隐藏类中优化分配空间的考虑 ?
https://developers.google.com/speed/articles/optimizing-javascript 
如果varA并不是在每个实例中都被初始化，那这样做将是有效果的。
*/
  doSomething : function(){
    // ...
  }
}

function B(a, b){
  A.call(this, a);
  this.varB = b;
}
B.prototype = Object.create(A.prototype, {
  varB : {
    value: null, 
    enumerable: true, 
    configurable: true, 
    writable: true 
  },
  doSomething : { 
    value: function(){ // override
      A.prototype.doSomething.apply(this, arguments); 
      // call super
      // ...
    },
    enumerable: true,
    configurable: true, 
    writable: true
  }
});
B.prototype.constructor = B;

var b = new B();
b.doSomething();
```
[原文链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)