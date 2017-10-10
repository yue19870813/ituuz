---
title: 使用TypeScript积累自己的类库
date: 2017-08-02 16:04:34
tags: 
	- TpyeScript
	- 类库
	- libs
categories: TpyeScript
---
>现在除了Web项目外，很多游戏引擎都支持js，包括Egret、Cocos、Unity等，甚至服务器端也可以用node.js。所以很多时候我们是可以积累一套通用工具库在不同项目间，甚至是不同类型项目、不同引擎间通用，来提高我们的开发效率。但是js的规范性较差，可维护性不强，有很多弊端，采用TypeScript来开发和积累我们的类库是比较好的一种选择。TypeScript是一种由微软开发的自由和开源的编程语言。它是JavaScript的一个超集，而且本质上向这个语言添加了可选的静态类型和基于类的面向对象编程。在易用性、可读性和易维护上都有了不小的提高。采用TypeScript来开发可以发布成js文件来使用。下面就简单介绍一下TypeScript的工作流。<!--more-->
## 安装TypeScript
通过npm（Node.js包管理器）来安装TypeScript:
```shell
> npm install -g typescript
```

## 创建第一个TypeScript文件
- 新建一个目录，在该目录下新建一个文件HelloWorld.ts
- 打开文件在文件内输入如下内容，涉及部分ts的语法就不讲了：
```typescript
class Test {
    private name:string = "Hello world";
    
    public constructor () {
        console.log("constructor==========>>>" + this.name);
    }

    public testFun():void {
        console.log("testFun===========>>>" + this.name);
    }
}

function testCallFun():void {
    let t = new Test();
    t.testFun();
}
```
- 编译ts文件
```
>tsc HelloWorld.ts
```
- 这样就将ts文件编译成了js文件，内容如下：
```javascript
var Test = (function () {
    function Test() {
        this.name = "Hello world";
        console.log("constructor==========>>>" + this.name);
    }
    Test.prototype.testFun = function () {
        console.log("testFun===========>>>" + this.name);
    };
    return Test;
}());
function testCallFun() {
    var t = new Test();
    t.testFun();
}
```
- 这里将这个js文件放到html页面上进行测试，控制台输出如下：
```
constructor==========>>>Hello world
testFun===========>>>Hello world
```
> 这样一套完整的工作流就完成了，生成的js文件可以用在各种项目中，Egret项目、cocos项目或者Web项目都可以。通过这种方式可以积累自己的类库，方便做项目时快速开发。

## 工程配置
### tsconfig.json
- 如果一个目录下存在一个tsconfig.json文件，那么它意味着这个目录是TypeScript项目的根目录。 tsconfig.json文件中指定了用来编译这个项目的根文件和编译选项。
- 在执行tsc时，编译器会在当前目录向父级目录逐级查找tsconfig.json文件，也可以使用命令行参数--project（或-p）指定一个包含tsconfig.json文件的目录
- 例子:
```javascript
{
    "compilerOptions": {
        "module": "system",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../built/ituuzx/itz.js"
    },
    "include": [
        "**/*.ts"   //include是指编译包含的文件或目录，这是配置的是包括子目录下的所有ts文件
    ]
}
```
>这样将开发过程中很多通用的问题抽象出来，封装成类库来使用，可以让开发效率更高，也让自己或者团队有技术沉淀。不同类型的类库可以封装到不同模块中去，分别解决不同问题，在使用时可以选择性的编译，只编译项目需要的模块，也方便管理。这种抽象和积累只会对个人或者项目有益，个人觉得是值得坚持的工作方法。后面我也计划会分享和维护一套游戏开发的工具类在github上，欢迎大家fork，地址是[https://github.com/yue19870813/ituuz-x](http://note.youdao.com/)，目前还没有提交代码，只是在README.MD里写了计划。
更多文章请关注我的公众号：
![这里写图片描述](http://img.blog.csdn.net/20170822194726017?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGV0dGhpbmtpbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)