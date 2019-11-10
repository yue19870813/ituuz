---
title: ituuz-x游戏框架v2.1特性介绍
date: 2019-11-10 20:36:42
tags: [cocos creator,js,typescript,框架]
categories: cocos creator
---

> v2.1版本主要是修复bug，以及对之前的功能进行了优化。

----
## v2.1版本功能列表
- [`new`]增加GameModel基类，目前增加了一些数据接口封装，是为了下个版本数据管理增加支持
- [`new`]View层的GameView增加onShow接口，该接口是view其他初始化结束后最终会调用的接口
- [`new`]ViewEvent增加注册点击事件，方便静态事件注册
- [`new`]Mediator增加customInit接口，该接口会在Mediator的init接口之前调用，通过该接口可对初始化过程进行干预
- [`new`]Mediator增加sceneContent属性，该属性是场景共享数据，在当前场景的所有Mediator中都可以读取该对象
- [`new`]Mediator的addView接口增加parent可选属性，可以自定义设置该view添加到的父节点
- [`new`]Mediator的addView接口增加useCache可选属性，来设置是否复用同类节点，默认false不复用
- [`bug`]修复android真机引起崩溃的问题
- [`bug`]修复Mediator的init和viewDidAppear接口调用顺序错误问题
- [`bug`]修改场景初始化生命周期异常问题
- [`bug`]修复全局场景层级缓存错误问题
- [`ts`]优化代码，增加注释，统一编码风格等  
<!--more-->  

## 主要功能详细介绍
- [`new`]ViewEvent增加注册点击事件，方便静态事件注册
```javascript
// v2.0之前注册点击事件如下：
let closeBtn = this.ui.getNode("close_button");
closeBtn.on(cc.Node.EventType.TOUCH_END, ()=>{
    this.closeView();
}, this);
// v2.1中增加了快捷注册方法：
// 只需要传入，节点名称，事件名称。和事件参数就可以了
// 需要注意的是，这种是静态注册，也就是说注册的时候参数就已经固定了，不能动态修改注册参数
// 如果需要在点击的时候动态传入参数，还需要用上面的那种方式
this.ui.addClickEvent("close_button", cc.Node.EventType.TOUCH_END, "close");
// 当然这种静态注册除了传入节点名称外，也可以直接把节点传入进行注册：
this.ui.addClickEvent(closeBtn, cc.Node.EventType.TOUCH_END, "close");
```
- [`new`]Mediator增加sceneContent属性，该属性是场景共享数据，在当前场景的所有Mediator中都可以读取该对象
```javascript
// 例如我们在DefaultSceneMediator场景中设置数据myName
public init(data?: any): void {
    this.sceneContent.data.myName = "ituuz";
}
// 然后我们在DefaultSceneMediator场景的FirstMediator层可以获取该共享数据：
public viewDidAppear(): void {
    // 获取当前场景的共享数据
    let myName = this.sceneContent.data.myName;
    // 设置给view在ui上显示出来
    this.view.setData(myName);
}
```
- [`new`]Mediator的addView接口增加parent可选属性，可以自定义设置该view添加到的父节点
```javascript
// 大多时候我们不需要关心弹出界面的父节点，这时我们直接调用addView即可：
this.addView(ViewCfg.POP_A_VIEW);
// 当你明确的想将打开的view界面添加到指定的节点时，则可以传入该节点，如下：
this.addView(ViewCfg.POP_A_VIEW, null, this.view.node);
// 上面就是将POP_A_VIEW打开并且设置其父节点为this.view.node
```
- [`new`]Mediator的addView接口增加useCache可选属性，来设置是否复用同类节点，默认false不复用
```javascript
// 打开界面B，并且设置第四个参数为true，意思是其节点复用，只会存在一个实例，而上面的界面B会存在多个实例
this.addView(ViewCfg.POP_B_VIEW, null, null, true);
// 可以在例子demo中尝试，可以反复打开多个界面A，但是界面B只有一个实例
// 当已经有个界面B在界面A的下面时，你再次打开界面B，则会复用这个实例，并将其移动到界面A的前面
// 而界面A会存在多个实例，打开几次就需要关闭几次
```
- [`bug`]修改场景初始化生命周期异常问题  
```javascript
public init(data?: any): void {
    // 此时view还没有初始化，不能对view进行操作
    // 在此处可以做mediator相关的初始化
}    
public viewDidAppear(): void {
    // 对view的操作需要在view初始化后才能进行，包括view的事件绑定
    this.view.setData("99999");
    this.bindEvent(FirstView.OPEN_A, (str: string)=>{
        this.addView(ViewCfg.POP_A_VIEW);
    }, this);
}
```

## 最后
以上就是本次版本更新的主要内容，详细使用可以参考例子demo，demo下载地址是:[https://github.com/yue19870813/ituuz-x/blob/develop/samples/LightmvcDemo_v2.1.zip](https://github.com/yue19870813/ituuz-x/blob/develop/samples/LightmvcDemo_v2.1.zip)