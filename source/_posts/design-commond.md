---
title: 设计模式在游戏开发中的应用之命令模式
date: 2016-11-29 14:13:45
tags: 命令模式
categories: 设计模式
---
设计模式在一些大型的软件系统中非常常用，用来处理复杂的结构和逻辑。游戏其实也是一个软件系统，也会有庞大的系统，复杂的逻辑关系，对设计模式的合理使用可以帮助我们更好的去组织各个系统模块，优化逻辑关系，使之可以更好的维护和拓展。本文对常用的设计模式在游戏中的应用进行讨论，而不对设计模式的原理进行过多的阐述了。本文的例子代码也是伪代码，不能够运行。<!--more-->

# 命令模式
## 通俗的定义  
将一组行为抽象为对象，使用不同的组合方式来执行命令，以实现解耦。本文介绍的命令模式可能与GoF上的稍有不同，是我自己对游戏开发中设计模式应用的理解。  
## 结构图：
![commond](/images/design_1_commond.png)
## 游戏开发中的使用
考虑以下场景，假如我们在设计一款RPG游戏，在野外地图肯定会有野怪，野怪会有一些AI逻辑，我们打算设计一套合理的怪物模块。大概如下：
``` java
//怪物基类  
class Monster {  
    //行走巡逻  
    public walk();  
    //攻击  
    public attack();  
    //逃跑  
    public escape();  
}
``` 大体设计包含了怪物行走、攻击和逃跑三个通用的行为特征。很快策划提出了新需求，要加入精英怪物类型，并且精英怪物有他自己独特的逻辑。嗯，幸好我们抽象出了怪物基类，只要继承过来，再增加新的行为即可，新增的精英怪物如下：
``` java
//精英怪物  
class EliteMonster extends Monster {  
    //精英怪物独有的行为  
    public eliteBehavior()  
}  
```
这样就很快的实现了新的需求，EliteMonster继承了Monster基础行为，并且增加了新的行为。接着没过多久策划又提出我们要有BOSS，是的，游戏怎么能没有BOSS呢，好吧我们来添加，毕竟我们设计好了基础行为，只要继承过来，在添加新的行为就好了：
``` java
//BOSS  
class BossMonster extends Monster {  
    //BOSS独有的行为  
    public bossBehavior()  
}  
```
看来我们的代码还是挺健壮的，每次都可以快速的增加新的怪物。但是紧接着新都修改需求提出来了：我们要让BOSS拥有精英怪物的行为。怎么办？还好，我们的程序足够健壮，修改一下继承关系就好了，我们让BossMonster来继承EliteMonster，虽然修改继承关系看起来很危险，但是我们还是完美的解决了问题，BOSS拥有了新的行为。经过一段时间，我们有设计了新的怪物：
``` java
//普通怪物  
class SimpleMonster extends Monster {  
    //普通怪物独有的行为  
    public simpleBehavior()  
} 
```
这时不幸的消息传来了：我们想让BOSS同时拥有精英怪物和普通怪物的行为！现在我们有几种解决方案：
- 把精英怪物的行为和普通怪物的行为，封装到Monster里，作为基础行为，但是这样的话，就是所有怪物都会继承这两种行为，这种多余的基础是我们不想看到的。
- 就是把SimpleMonster的行为复制一份给BOSS，让BOSS在继承EliteMonster的同时拥有SimpleMonster的行为，但是这样就会有代码的冗余，后面我们修改这种行为的时候就要在两个地方修改。
- 期望你使用的语言可以多继承。然而多继承并不是一个好的特性和方案。

这时我们想一想，算了，代码冗余就冗余一下吧，毕竟大多数需求我们还是可以很好的实现的。就在我们觉得可以满足需求时，更糟糕的需求又提交过来了：我们需要挂机功能，玩家的角色需要实现一部分AI功能，这时就麻烦了，我们怎么调整这个继承关系，让Player继承谁？看起来继承谁都不太合理。这时我们就要思考一下到底该怎么设计行为这部分，才能让我们适应各种需求的改动。看一下下面这种设计：
``` java
//游戏中行为对象的基类  
class GameActor {  
}  
//玩家对象  
class Player extends GameActor {  
  
}  
//命令抽象接口  
interface ICommond {  
    public execute(GameActor ga);  
}  
//攻击命令  
class AttackCommond {  
    //参数可以是任何一个行为对象  
    public execute(GameActor ga);  
}  
//执行命令的类  
class CommondInvoke {  
    public addCommond(ICommond commond);  
    public execute();  
}  
//创建一个玩家对象  
Player p = new Player();  
//创建一个攻击命令  
ICommond attackCommond = new AttackCommond(p);  
//让玩家执行攻击命令  
CommondInvoke invoke = new CommondInvoke();  
invoke.addCommond(attackCommond);  
invoke.execute() 
```
行为对象都继承GameActor，可以随时增加新的对象。行为命令需要传入一个行为对象来执行行为动作，这里命令与对象进行了解耦，命令与对象可以随意增加与组合。CommondInvoke也可以进一步优化，可以执行多条命令，可以倒序或顺序执行，可以同步或异步，甚至可以随时添加、删除和修改。多个命令的组合也是命令模式的一个特点。此外命令模式还可以处理事务回滚：
``` java
//命令抽象接口  
interface ICommond {  
    public execute(GameActor ga);  
    //撤销函数  
    public undo();  
}  
```
事务回滚这个特性在服务器中比较常用，比如玩家的一次购买行为，肯定是要保障金钱扣除，道具添加同时打成，否则就算是事务失败，要把修改的内容还原，这时就可以调用undo进行回滚。命令模式暂时介绍这些，下一篇会介绍观察者模式。

