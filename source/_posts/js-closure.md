---
title: JS闭包总结
date: 2017-02-02 16:04:34
tags: 闭包
categories: javascript
---
## 闭包的用途
#### 1.防止全局变量污染
在JavaScript中全局变量是个不小的毒瘤，全局变量有时是很方便，但是很多项目滥用全局变量成灾，维护起来非常困难。所以这里的作用就是防止全局变量污染，例子如下：
```javascript
var add = (function() {    
    var i = 0;   
    return function add () {       
        i++;        
        cc.log(i); // 1    
    }
})();

add();
```
<!--more-->
这个例子实现的功能就是，减少了全局变量的滥用，同时这个功能也是达到了下面的用途，就是访问局部变量。
#### 2.局部变量访问
上面防止全局变量污染的例子中i是一个局部变量，但是在函数外依然可以间接的访问控制，就是达到了在作用域外访问局部变量。
#### 3.匿名初始化
匿名初始化很简单，有时我们需要一个界面打开做一些初始化，只需要执行一遍，可以采用这种形式：
```javascript
(function () {    
    cc.log("init something...");
})();
```
这样在加载时直接就执行了匿名函数里的东西，有效的防止了，一些在不知情的情况下重复的调用了初始化函数。
#### 4.私有成员封装
这个是比较常用的，JavaScript中并没有很多语言中拥有的访问控制，所以我们可以使用闭包来达到私有化的目的：
```javascript
var person = (function () {    
    var age = 18;    
    return {        
        getAge:function () {            
            return age;       
        },        
        setAge:function (a) {           
            age = a;        
        }    
    }
})();

cc.log(person.getAge()); // 18
person.setAge(99);
cc.log(person.getAge()); // 99
```
这样在外部就不能访问age变量，只能通过getAge和setAge来访问，达到了私有化和封装的目的，这点在开发中比较常用。
#### 5.制作缓存池
这个用途是使用了闭包里的局部变量不会被销毁的特点，实现缓冲池的方法有很多，使用闭包只是其中一种，大家可以看自己的情况来使用，下面是使用闭包来实现缓存池简单大意的例子：
```javascript
var person = (function () {    
    var pool = null;    
    return {        
        getObj:function (key) {            
            return pool[key];        
        },        
        setObj:function (key, obj) {            
            pool[key] = obj;        
        }    
    }
})();
```
#### 6.循环中保存索引
开发过程中在循环里注册回调很常见，但是刚接触js的同学常常会遇到注册完了回调，发现索引值不对，可以看下面的例子：
```javascript
var arr = [];
var show = function () {    
    for (var i = 0; i < 3; i++) {        
        arr[i] = function test () {            
            cc.log(i);        
        }    
    }
}

show();

arr[0](); // 3
arr[1](); // 3
arr[2](); // 3
```
运行会发现，输出的都是最后一个索引，这种情况是因为，内部函数并没有立即执行，而且内部函数中只是对这个变量进行了引用，所以最后记录的都是这个引用地址中的最终值。要想得到我们想要的效果就需要让内部函数立即执行，捕捉当时索引值，可做如下改动：
```javascript
var arr = [];
var show = function () {    
    for (var i = 0; i < 3; i++) {        
        arr[i] = (function (j){            
            return function () {                
                cc.log(j);            
            }        
        })(i);    
    }
}

show();

arr[0](); // 0
arr[1](); // 1
arr[2](); // 2
```
这样就获得了我们想要的结果。  
#### 7.闭包的其他问题  
- 因为闭包中的局部变量不会释放，所以闭包和全局变量一样会占用大量内存。
- 闭包中变量引用由于不会释放，很有可能造成内存泄露。
- 大量的闭包使用可能会降低程序的可读性，增加维护成本。

总之闭包有很多地方用起来很方便，闭包的特性也能帮助我们实现很多巧妙的设计，但是闭包也会引起很多问题，所以在开发过程中应该尽量少用闭包。