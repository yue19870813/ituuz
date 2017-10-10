---
title: JavaScript性能优化总结
date: 2017-03-06 23:14:36
tags: javascript,性能优化
categories: javascript
---
### JavaScript性能优化总结
- +=运算符比+运算符效率高。
- 数字转换成字符串
```
("" +) > String() > .toString() > new String()
```
- 展开循环：当循环次数是确定的，消除循环并使用多次函数调用，虽然会让代码看起来不够高端，但是速度往往会更快。
- 减值迭代:在循环中，减值要比增值效率高。
```
//增值迭代
for (var i = 0; i < 1000; i++) {
    //TODO ...
}
//减值迭代较优
for (var i = 999; i >=0; i--) {
    //TODO ...
}
```
- 使用switch优于if：在分支大于2并且允许使用switch的时候用switch的效率是很高的，因为switch是随机访问的。
- 重复值:任何在多处用到的值都应该抽取为一个常量
```
var result = (obj.value.value2 + obj.value.value3)*obj.value.value1;
//较优的写法
var value = obj.value;
var result = (value.value2 + value.value3)*value.value1;
```
- 遍历数组时，缓存数组长度
```
var len = arr.length;
for (var i = len - 1; i > 0; i--) {
    //TODO ...
}
```
- 释放对象：  
```
//对象
obj = null  
对象属性：delete obj.myproperty  
数组item：使用数组的splice方法释放数组中不用的item  
```
- 位运算较快：&、|、!、>>、<<
```
//按位与(&):判断一个数是奇数还是偶数
if (n & 1) {
    console.log("n是奇数");
} else {
    console.log("n是偶数");
}

//按位或(|):对浮点数向下求整
var num = 1.1 | 0; // 1

//左移(<<) 
var num = 2 << 1; // 4
//右移(>>)
var num = 64 >> 1; // 32
```
- 巧用||和&&布尔运算符
```
if (myobj) {
    doSomething(myobj);
}
//可以替换为：
myobj && doSomething(myobj);
```
- 避免与null进行比较
```
if (obj == null) {
    //todo
}
//替换为
if (!obj) {
    //todo
}
```





