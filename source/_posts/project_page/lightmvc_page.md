---
title: ituuz-x
date: 2019-07-29 15:14:08
permalink: ituuz-x
tags: [typescript, ituuz-x, cocos框架]
categories: ituuz-x
---
ituuz-x项目主页
---
ituuz-x是一个cocos creator游戏开发一个集成框架，也是一个工具集。其包括常用用的项目管理的mvc架构，以及静态数据、本地化、资源管理、网络等模块，后续还会不断拓展新的模块或工具，除核心`core`是必须引用外，其他模块都可以根据需求选择引用，后续会增加模块剔除配置功能。框架后续功能可以参考下面的Planned计划表。
<!--more-->

### 入门介绍
[lightMVC模块介绍:轻量级游戏开发mvc框架](http://ituuz.com/2019/07/15/lightMVC-1/)

### 近期版本内容
#### v2.0新增功能（lightMVC_ex）
> v2.0主要对核心模块中的lightMVC进行了拓展，增加了更多接口和功能，方便更大规模项目使用。
> 相关文档：待补充
- 框架全局可调用的接口
    - 框架初始化
    - initScene：初始化第一个场景
- 新增GameMediator基类
    - runScene
    - popView
    - addLayer
- 新增GameView基类，对应GameMediator，暂无新增功能。
- 图集自动加载功能


#### v1.0版本
> 轻量级的mvc框架，相关文档[lightMVC模块介绍:轻量级游戏开发mvc框架](http://ituuz.com/2019/07/15/lightMVC-1/)
- lightMVC核心模块基本功能

