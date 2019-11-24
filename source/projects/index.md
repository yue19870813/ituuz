---
title: 游戏开发框架：ituuz-x
date: 2019-07-29 15:14:08
tags: [typescript, ituuz-x, cocos框架, creator]
categories: ituuz-x
---
ituuz-x项目主页
---
ituuz-x是一个cocos creator游戏开发一个集成框架，也是一个工具集。其包括常用用的项目管理的mvc架构，以及静态数据、本地化、资源管理、网络等模块，后续还会不断拓展新的模块或工具，除核心`core`是必须引用外，其他模块都可以根据需求选择引用，后续会增加模块剔除配置功能。框架后续功能可以参考下面的Planned计划表。
<!--more-->

### 入门介绍
[core-mvc模块介绍:轻量级游戏开发mvc框架](http://ituuz.com/2019/07/15/lightMVC-1/)
[mvc_ex模块介绍:mvc拓展模块](http://ituuz.com/2019/10/09/mvc-ex/)
[ituuz-x游戏框架v2.1:新特性介绍](http://ituuz.com/2019/11/10/ituuz-x-v2-1特性介绍/)
[ituuz-x游戏框架v2.2:网络核心模块](http://ituuz.com/2019/11/14/ituuz-x-net/)

### 近期版本内容
#### v2.2版本功能
> v2.2版本主要是对网络核心模块进行了封装，完善了框架体系。这个版本中目前只对http进行了实现，其他类型都是未实现的接口。相关文档：[ituuz-x游戏框架v2.2:网络核心模块](http://ituuz.com/2019/11/14/ituuz-x-net/)
- [`new`]网络核心模块`net`
- [`new`]GameModel增加网络通信相关接口
- [`plug-in`]增加creator插件pb-generator
- [`bug`]修改Mediator销毁接口destroy有时不会调用的bug
- [`bug`]修改场景切换时，部分数据没有销毁的bug

#### v2.1版本功能列表
> v2.1版本主要是修复bug，以及对之前的功能进行了优化。相关文档：[ituuz-x游戏框架v2.1:新特性介绍](http://ituuz.com/2019/11/10/ituuz-x-v2-1特性介绍/)
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

#### v2.0新增功能（lightMVC_ex）
> v2.0主要对核心模块中的lightMVC进行了拓展，增加了更多接口和功能，方便更大规模项目使用。相关文档[mvc_ex模块介绍:mvc拓展模块](http://ituuz.com/2019/10/09/mvc-ex/)
- 框架全局可调用的接口
- 新增GameMediator基类
- 新增GameView基类，对应GameMediator，暂无新增功能。
- 图集自动加载功能

#### v1.0版本
> 轻量级的mvc框架，相关文档[lightMVC模块介绍:轻量级游戏开发mvc框架](http://ituuz.com/2019/07/15/lightMVC-1/)
- lightMVC核心模块基本功能

[github地址：https://github.com/yue19870813/ituuz-x](https://github.com/yue19870813/ituuz-x)


