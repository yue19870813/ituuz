---
title: cocos2d-x游戏中的性能优化和内存优化
date: 2017-01-06 23:14:36
tags: cocos2d-x,性能优化
categories: cocos2d-x
---
在游行开发的中后期所有的团队基本都会遇到两个通性问题：卡顿和崩溃。造成卡顿也就是迪奥帧的原因主要是CPU计算量和GPU渲染压力过大。造成程序崩溃的原因基本也是两种情况，一种是代码错误造成的崩溃，另一种是我们主要讨论的内存过高造成的崩溃。下面就对游戏中遇到的这两个问题进行讨论，讨论的游戏主要是针对2D游戏，当然很多情况是2D和3D游戏中共同存在的。<!--more-->

- __载入纹理时按照从大到小的顺序，并且分帧加载。__因为cocos2dx中加载一张纹理，要占用两倍的纹理空间，多余的内存会在这帧结束时释放掉，所以从大到小，分帧加载的策略可以高效的使用内存防止内存峰值过高造成崩溃。需要注意的是避免使用超大纹理。
- __资源格式的选择。__主要有3个因素会影响纹理内存，即纹理格式（压缩还是非压缩）、颜色深度和大小。我们可以使用PVR格式纹理减少内存使用。推荐纹理格式为pvr.ccz。纹理使用的每种颜色位数越多，图像质量越好，但是越耗内存。所以我们可以使用颜色深度为RGB4444的纹理代替RGB8888，这样内存消耗会降低一半。如果图片不需要透明通道就不要加上透明通道，这样还可以节省内存的使用。同时pvr格式的纹理在上传到gpu时的速度也要比其他格式更快一些。
- __减少绘制调用和上传gpu压力。__首先纹理需要上传到gpu中渲染绘制，越小的纹理上传的速度越快，所以要控制避免有较大的纹理一次上传到显存中，上面说过pvr格式的纹理上传速度会快些，同时大量的纹理上传会造成程序瞬间卡顿，因为纹理上传是占用主线程的，所以低一点中的分帧加载在这里也适用。还有就是绘制的压力，同屏中显示对象的数量会影响绘制的性能，所以在合理的情况下要尽量减少显示对象的数量。同时我们要尽量将散图打到一张整图中，因为在cocos2dx引擎的底层中，在同一张纹理上的显示对象只会调用一次绘制命令，相当于BatchNode。这里需要特别注意的是减少绘制要打到整图中和为了加快纹理上传速度要减小纹理大小是个相悖的条件，所以我们要根据实际情况来衡量这两点。一般情况下每帧上传一张1024*1024的rgba8888格式的纹理是不会出现卡顿的，所以建议每次上传不要超过这个大小。如果是rgb4444的纹理那么大小会减少一半，上传会更快一些。
- __降低UI复杂度。__降低复杂度有很多方式，第一种方式是减少显示对象数量，比如拿文本框来说，经常在游戏中会有显示金币的地方，例如：“金币：99”。有很多人在ui里会将标题和数量分开，定义两个文本框，这里如果没有特殊表现的话，就应该合并成一个文本框，这样就减少了一个对象的绘制。同理有很多东西都可以减少，例如Sprite等，项目中具体情况具体对待。第二种方式是降低UI树的复杂度，上一点中减少数量也会达到一个降低UIU树复杂度的效果，在这个同时，我们还要尽量降低UI树的深度，降低树的深度可以有效的提高绘制遍历的速度，在设计ui的时候，我们往往为了易于管理，会创建很多空面板，或节点，来达到管理ui中各个部件的作用，但是有时我们会增加的过于复杂，这里尽量降低这种复杂度就可以提高绘制效率。最后我们还要控制资源的规格，首先游戏中很多ui是通用的，比如通用的按钮，图标，面板等，我们打到一张贴图上，作为通用资源常驻内存，然后每个ui独特的资源单独打成贴图，在使用的时候加载到内存中，使用结束后释放掉。
- __UI设计的时候要提高UI通用资源的使用率。__优化资源到最小，比如使用九宫格图片，采用旋转缩放等丰富资源表现力等。
- __动画文件的处理。__动画一般有两种选择，帧动画和骨骼动画。帧动画的优点就是运算不耗性能，并且表现较好，缺点就是当帧数较多时，资源量比较大，并且占用内存较高，所以不太适合做过于复杂的动画，比如动作和技能较多的ARPG角色如果纯用帧动画来实现，会使用很多资源，在加载时就不能加载过多的角色，这样就局限了游戏的设计，同场景角色数量不能过多。与之相对应的就是骨骼动画，骨骼动画的特点就是节省资源，一套骨骼可以做很多套动作，新增动作也不会对文件大小有明显的增加，缺点也比较明显，就是因为骨骼动画是通过计算，然后进行旋转，位移等实现的，所以当动画比较复杂，骨骼数量较多时，计算量就会很大，对CPU的计算增加了压力，压力过大时就会掉帧卡顿。当团队技术较好，并且美术程序沟通配合默契的团队可以采用帧动画与骨骼动画相结合的方案。美术在设计时动作大部分采用骨骼，局部需要表现效果的地方使用帧动画到处的贴图来实现，美术人员在设计时要充分考虑到资源利用，利用旋转缩放功能丰富动画，同时还要考虑骨骼数量、节点深度等队性能等影响。
- __游戏音效的规格及优化。__游戏音效在使用中也有很多问题，音效占用的资源空间、运行时内存和加载效率等。首先需要考虑使用资源的格式，在各个平台下，资源格式的支持也都不同，平台及支持的常见文件格式如下：  
Android ：mp3, mid, oggg, wav  
iOS ：mac, caf, mp3, m4a, wav  
Windows ： mid, mp3, wav  
通常我在项目中会针对不同平台加载不通资源，一般android常用oggg，iOS中常用caf，windows下一般都是调试用的，也没怎么优化过，直接用的mp3格式。同时音效资源也需要处理，音频优化,3个因素会影响音频文件的内存使用，即音频文件数据格式、比特率及采样率。推荐使用MP3数据格式的音频文件，因为Android平台和iOS平台均支持MP3格式，此外MP3格式经过压缩和硬件加速。背景音乐文件大小应该低于800KB，最简单的方法就是减少背景音乐时间然后重复播放。音频文件采样率大约在96-128kbps为佳，比特率44kHz就够了。
- __游戏逻辑的相关优化。__在代码中，不要在主循环中做复杂的逻辑处理，例如for循环更新、查找等，可以利用缓冲等技术，解决和优化在主循环中每帧的效能。部分不重要的逻辑，或者要求即时反馈不高的逻辑可以适当的采取抽帧，也就是不是每帧都进行运算和刷新，这样可以降低很多运算量。同时游戏的玩法逻辑也有很多可以优化的地方，比如射击游戏的子弹可以做缓存，减少频繁的创建和销毁对象；RPG游戏中的地图可以做成异步分块加载，不显示的可以不去加载渲染等。
- __代码级别的优化。__这点比较简单，在编写程序时，算法最优，并且不要有内存泄漏等。
- __最后就是引擎级别的优化。__我们使用的是cocos2dx进行的项目开发，在该引擎中，存在的bug、性能等问题也有很多可以优化的地方。在我们项目中，就修改过spine的内存泄漏问题、ui加载速度问题、还有关于对象拷贝的性能问题等。修改引擎要对引擎比较了解，并且修改后引擎的版本再次升级是还要把修改合并过去，这里也建议大家发现问题可以到引擎的Github上提交request。如果对渲染比较了解的还可以对渲染级别进行优化。



