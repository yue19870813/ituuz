---
title: cocos探照灯效果实现
date: 2020-06-11 14:18:36
tags: [cocos creator,探照灯,光照,shader,js]
categories: cocos creator
---

## 介绍
探照灯效果就是指整个场景或者图片都是黑的，只有灯光照射的地方才是亮的。实现方式有很多种，我们这里用shader来实现，主要原因就是用shader来实现，效率更高，效果更好，并且拓展性更强一些。下面是一个探照灯效果：
![探照灯效果](/images/searchlight.gif)
<!--more--> 

## 探照灯实现
### 光照原理
我们知道在程序中颜色由RGB三个数值来决定，比如红光的RGB值是255、0、0，但在GLSL语言中表示颜色的最大值是1.0，而且通常用一个vec4来表示，所以红光就是：vec4(1.0, 0.0, 0.0, 1.0); 第四个参数是alpha值，这里我们都设置为1。  

这里我们再说一下现实中我们看到物体有各式各样的颜色，并不是它们本身是这个颜色，而是它所能反射光的颜色。太阳光中是包含所有颜色光的，而一个物体能反射什么颜色的光与物体本身粒子特性有关，决定了它能反射光的波长范围，而其他范围的光则被吸收，所以我们就看到了各种颜色的物体。如下图所示：  
![光照颜色原理](/images/light_fanshe.png)    

利用这个原理，我们就可以通过shader来控制目标的颜色显示来模拟光照了，比如我们设定一个白色光照：vec4(1.0, 1.0, 1.0, 1.0)。该向量与片段着色器中的颜色相乘，因为每个值都是1，所以物体颜色并不会改变。当我们设置为红光vec4(1.0, 0.0, 0.0, 1.0)时，物体就会发红。  

我们再控制光照的范围是一个圆形，那么就是一个探照灯效果了。

### 着色器实现
- 下面是着色器的实现过程，不是完整代码，只有代码片段，熟悉GLSL语言的同学应该可以看懂，不了解的同学可以直接到文末获取完整代码再回头过来学习亦可。
- 首先是光照范围逻辑，根据两点距离和光照中心点来判断是否在光照范围内，光照范围内的点为目标颜色，范围外的点则设置成黑色：
```c++
// 计算当前点是否在光照范围内
float dist = sqrt(pow(fragPos.x - light_center.x, 2.0) + pow(fragPos.y - light_center.y, 2.0));
// 光照半径
float light_radius = 0.2;
// 根据点到光源圆心的距离判断光照范围
if (dist > light_radius) {
    // 超出光照范围则为黑色
    gl_FragColor = vec4(0, 0, 0, 1.0);
} else {
    gl_FragColor = o;
}
```
- 运行效果：  
![circle_light](/images/circle_light.jpg)
- 此时一个大体的光照范围效果就有了，但是不太自然，光照的边缘没有过渡，并且有锯齿存在，下面我们通过插值来把光照的边缘增加过渡效果：
```c++
// 通过插值让光照过度平滑
float r = (1.0 - smoothstep(light_radius, light_radius + 0.1, dist));
gl_FragColor = vec4(o.r * r, o.g * r, o.b * r, 1.0);
```
- smoothstep函数接收三个参数，第一个是范围的最小值，第二个是范围的最大值，第三个参数是需要计算的目标值x，x小于最小值则返回0，大于最大值返回1，其余返回0到1的插值。上面代码里的范围是光照半径及半径加0.1的范围内进行插值来实现光线边缘的过渡效果。修改完再看，效果已经不错了。  
![circle_light_smooth](/images/circle_light_smooth.jpg)
- 最后我们通过代码来设置light_center参数就能实现光源的移动了：
```javascript
// Touch move事件函数
public touchMove(event: cc.Event) {
    let touch: cc.Touch = event.touch;
    // 根据屏幕坐标获得-1 ～ 1 的范围坐标，传递给着色器使用
    let x = touch.getLocation().x / (cc.winSize.width / 2) - 1;
    let y = touch.getLocation().y / (cc.winSize.height / 2) - 1;
    let center = cc.v2(x, y);
    this._materi.effect.setProperty("light_center", center);
}
```
- 细心的朋友可能发现，如果你的场景或者目标图片不是正方形，那么光照的范围可能就不是个正圆了，而是一个椭圆。这是因为OpenGL中的坐标范围是从-1到1，但宽高长度并不一定是等比的。所以我们在计算圆的范围时，可以把场景或者图片的宽高比传入进来，然后计算圆范围时进行修正就可以了：
```c++
// 计算当前点是否在光照范围内, 增加宽高比的修正，wh_ratio是传入的目标宽高比。
float dist = sqrt(pow(fragPos.x * wh_ratio - light_center.x * wh_ratio, 2.0) + pow(fragPos.y - light_center.y, 2.0));
```

## 进一步优化
### 漫反射环境光
- 大家知道，无论我们在多么漆黑的屋子里，我们总是能隐约看见物体轮廓的，这就是因为有漫反射的环境光造成的。物体表面都不是光滑的，都是凹凸不平的，总会把光源的光线，反射到各个方向，其它物体接收到反射光后会再次反射，反复循环，这就是漫反射，我们也叫环境光。这就是为什么我们在没有光源的屋子里也能看见东西的原因，漫反射原理如下图：  
![漫反射原理](/images/light_mfs.jpg)
- 在我们的例子中光线没有照到的地方漆黑一片，有时我们也需要一些漫反射造成的环境光。真实的漫反射算法非常复杂，要通过法向量来运算，我们这里只是一个2D的纹理贴图，所以我们就把环境光设置成一个平均的常数就可以了，在光线没有照到的逻辑中加入环境光逻辑：
```c++
// 设置环境光强度为0.1
float ambient_strength = 0.1;
// 通过插值让光照过度平滑
float r = (1.0 - smoothstep(light_radius, light_radius + 0.1, dist));
// 该判断用于保证漫反射的环境光有效
if (r <= ambient_strength) {
    r = ambient_strength;
}
gl_FragColor = vec4(o.r * r, o.g * r, o.b * r, 1.0);
```
- 这时运行效果就和教程开头的效果基本一样了。

### 光照强度和光源颜色
- 前面的实现过程中我们对在光照范围内的目标颜色没有干预，保证了目标的固有色。但有时我们也需要改变一些，比如上面讲到了环境光，没有光照的地方并不是漆黑一片了，那么反过来，有光照的地方就一定是亮的吗？有时我们也需要调整一下光照的亮度和光照的颜色。
- 我们先看光照的亮度，这个比较简单，逻辑和环境光强度一样，我们再定义一个光源强度的参数就可以了：
```javascript
// 光照强度，我们修改为0.8
float result = 0.8;
// 根据点到光源圆心的距离判断光照范围
if (dist > light_radius) {
    // 设置环境光强度为0.1
    float ambient_strength = 0.1;
    // 通过插值让光照过度平滑
    float r = (1.0 - smoothstep(light_radius, light_radius + 0.1, dist)) * result;
    // 该判断用于保证漫反射的环境光有效
    if (r <= ambient_strength) {
        r = ambient_strength;
    }
    gl_FragColor = vec4(o.r * r, o.g * r, o.b * r, 1.0);
} else {
    gl_FragColor = vec4(o.r * result, o.g * result, o.b * result, 1.0);
}
```
- 我们将光照部分和参与插值运算的光照边缘过渡部分都乘上光照强度就可以实现我们的目标效果了。（不过这里需要注意的一个问题是，光照强度result如果小于环境光强度ambient_strength会是什么效果呢？大家可以试一下，并尝试修改。）
- 接下来我们增加光源颜色的支持，根据教程开头讲的光照原理，我们可以直接把之前我们例子中的目标颜色乘以一个光源颜色就可以了，光源颜色我们就用上面说过的红光：
```javascript
// 光源颜色,红光
vec4 light_color = vec4(1, 0, 0, 1);
// 光照强度，我们修改为0.8
float result = 0.8;
// 根据点到光源圆心的距离判断光照范围
if (dist > light_radius) {
    // 设置环境光强度为0.1
    float ambient_strength = 0.1;
    // 通过插值让光照过度平滑
    float r = (1.0 - smoothstep(light_radius, light_radius + 0.1, dist)) * result;
    // 该判断用于保证漫反射的环境光有效
    if (r <= ambient_strength) {
        r = ambient_strength;
    }
    gl_FragColor = vec4(o.r * r, o.g * r, o.b * r, 1.0) * light_color;
} else {
    gl_FragColor = vec4(o.r * result, o.g * result, o.b * result, 1.0) * light_color;
}
```
- 运行效果如下：  
![最终效果](/images/searchlight_color.gif)

## 最后
[完整代码地址](https://github.com/yue19870813/cocos_demo/tree/master/CocosDemo_2_3_3/assets/case/searchlight)，如果有其它问题可以关注我的公众号给我留言。
