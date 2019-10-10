---
title: ituuz-x框架v2.0新增mvc拓展模块[mvc_ex]
date: 2019-10-09 14:18:31
tags: [typescript, javascript, js, ituuz-x, cocos框架, mvc]
categories: ituuz-x
---

本节内容是基于[ituuz-x](http://ituuz-x.ituuz.com/)框架[core中的mvc](http://ituuz.com/2019/07/15/lightMVC-1/)做的拓展，对mvc中基本接口进行了封装，更加方便规模较大项目进行开发和维护，v2.0中新增加的内容如下：  
- 框架全局可调用的接口
    - 框架初始化
    - initScene：初始化第一个场景
- 新增GameMediator基类
    - gotoScene：运行新场景
    - addView：打开新UI界面
- 新增GameScene和GameView基类，对应GameMediator。
- GameView增加图集根据UI模块自动加载功能。
<!--more-->

### 封装框架初始化接口
```javascript
/**
 * 初始化框架接口
 * @param {MVC_struct} scene 初始化游戏的第一个场景
 * @param {boolean} debug 框架是否是调试模式
 * @param {cc.Size} designResolution 默认的设计分辨率
 * @param {boolean} fitWidth 是否宽适配
 * @param {boolean} fitHeight 是否高适配
 */
public static start(
        scene: MVC_struct,
        models: {new (): BaseModel}[],
        debug: boolean = true, 
        designResolution: cc.Size = cc.size(960, 640), 
        fitWidth: boolean = false, 
        fitHeight: boolean = false
    ): void;
```
该接口是在游戏启动时第一时间调用的，作为框架初始化的首要接口。在该接口中重要的是前两个参数，第一个参数是一个MVC_struct类型的结构体，是框架初始化的第一个运行场景。该结构体如下：
```javascript
export class MVC_struct {
    public viewClass: string; // view类名
    public medClass: string;    // mediator类名
    public children: MVC_struct[]; // 子节点
}
```
从上面结构中可以看到，该结构体是对一个模块的封装，由视图层view类名，控制层mediator类名构成，同时还支持子节点嵌套，创建场景或者View时会按照该结构体的节点关系进行创建。在mvc_ex中，运行场景或者打开View模块都是需要传入一个[MVC_struct]结构体作为参数，所以我们业务代码中可以统一在一个文件中定义这些结构体并配置这些模块，方便业务对所有场景或者View模块统一管理和维护。例如我们可以单独在一个文件中做如下配置：
```javascript
public static readonly MAIN_SCENE: MVC_struct = {
    viewClass: "MainScene",
    medClass: "MainMediator",
    children: [
        {
            viewClass: "UIView",
            medClass: "UIMediator",
            children: [
                {
                    viewClass: "OpratorView",
                    medClass: "OpratorMediator",
                    children: []
                }
            ]
        },
        {
            viewClass: "PlayerView",
            medClass: "PlayerMediator",
            children: []
        }
    ]
};
```
这样我们可以随时调节各个视图层view的节点关系，同时游戏场景的节点关系也一目了然，哪些类对应哪些模块，在什么层级也都清晰可见。

### GameMediator基类
GameMediator基类中新增gotoScene：运行新场景和addView：打开新UI界面两个接口，这两个接口中的第一个参数都是MVC_struct结构体，和上面初始化接口一样，可以根据结构体初始化场景或者View；这两个接口的第二个参数是可选的any类型的参数，可以用来将参数透传给要打开的场景或者view。

### GameScene和GameView基类
这两个基类分别对应场景的视图层和UI的视图层。GameView中新增了两个接口atlasName和setSpriteFrame这两个接口一个是下面要说到的自动加载该模块纹理贴图功能，另一个是使用加载好的纹理设置sprite。

### View模块自动加载图集功能
上面说到GameView基类新增了个atlasName接口，该接口就是用来配置该模块需要加载的图集资源，声明如下：
```javascript
/**
 * 子类重写来实现资源的加载和释放
 */
public atlasName(): string[] {
    return [];
}
```
return的数组中用来配置该模块需要加载的图集资源。例如：
```javascript
/**
 * 子类重写来实现资源的加载和释放
 */
public atlasName(): string[] {
    return [
        "atlas1",
        "atlas2"
    ];
}
```
这样"atlas1"和"atlas2"两个纹理图集就会自动加载了，然后在view中可以直接调用setSpriteFrame接口来为sprite设置纹理信息，该接口的声明如下：
```javascript
/**
 * 为sprite设置纹理
 * @param {cc.Sprite} sprite sprite对象
 * @param {string} name 纹理名称
 * @param {string} key 图集key值，如果不传key，则默认使用该UI的第一个图集来设置；
 */
public setSpriteFrame(sprite: cc.Sprite, name: string, key?: string): void;
```

### 结语
以上就是这次ituuz-x框架更新的内容，有问题可以直接留言反馈或者给我发邮件，如果感到有用就关注我的公众号吧。

[github地址：https://github.com/yue19870813/ituuz-x](https://github.com/yue19870813/ituuz-x)