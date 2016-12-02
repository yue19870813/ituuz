---
title: 设计模式在游戏开发中的应用之观察者模式
date: 2016-12-01 22:41:17
tags: 观察者模式
categories: 设计模式
---
# 观察者模式
## 通俗的定义
触发事件的一方不关心谁来处理，处理事件的一方不关心事件是从哪里来的。观察者模式就是让观察者与被观察者彻底解耦。
## 结构图如下
![observer](/images/design_2_observer.png)
<!--more-->
## 游戏开发中的使用
当我们设计一个成就系统的时候，往往要在各个系统都要增加判断，比如杀死某种怪物多少只，新手往往可能这么写：
``` java
public static KILL_1009_COUNT = 0;  
main () {  
    //killMonster是杀死一个怪物，返回杀死怪物的id  
    int id = killMonster();  
    //如果杀死的怪物是1009那么数量加一，超过100次达成成就  
    if (id == "1009") {  
        KILL_1009_COUNT++;  
        if (KILL_1009_COUNT > 100) {  
            print("达成成就！杀死1009怪物100次！");  
        }  
    }  
}  
```
如果这样写下去，后果将不堪设想：各个类直接将会超级耦合，成就判断将会蔓延到整个项目的每个角落！观察者模式就是为了解决这个问题而出现的。观察者模式让代码彻底解耦，还是上面的那个例子：
``` java
main () {  
    //killMonster是杀死一个怪物，返回杀死怪物的id  
    int id = killMonster();  
    //发送消息，杀死怪物，并将杀死的怪物id一起发送  
    notify(EVENT_KILL_MONSTER, id);  
}  
```
这样代码的各个功能就不用关心成就相关的逻辑，只是通知我做了这样一件事情就可以了。同样，游戏中这样的例子到处都是，比如增加经验时，我们发送增加经验的消息，接收消息的地方来处理到底升没升级，因为可以增加经验的地方有很多，这样我们就不用导出判断是否升级了。下面看一下观察者的实现：
``` java
//处理观察者与被观察者  
class Notification {  
    //一个可变的观察者列表  
    private List observerList = null;  
    //该类是单例（后面我们会单独讲单例模式）  
    private static Notification _instance = null;     
    public static getInstance() {  
        if (_instance == null) {  
            _instance = new Notification();  
            //初始化列表容器  
            observerList = new ArrayList();  
        }  
        return _instance;  
    }  
    //添加观察者  
    public void addObserver(Observer obs) {  
        if (!observerList.isExistobs {  
            observerList.add(obs);  
        }  
    }  
    //删除观察者  
    public void removeObserver(Observer obs) {  
        observerList.remove(obs);  
    }  
      
    //广播消息  
    public void sendMsg(String event, ..) {  
        for (Observer obs : observerList) {  
            if (obs.event == event) {  
                obs.onNotify(..);  
            }  
        }  
    }  
}  
  
//观察者基类  
class Observer {  
    String event;  
    //接收消息函数，event是消息类型，后面是参数  
    void onNotify(..)  
    //添加观察者  
    void addObserver(String event) {  
        event = event;  
        Notification.getInstance().addObserver(this);  
    }  
}  
  
//杀怪成就观察者  
class AchievementObserver extends Observer {  
    void onNotify(id) {  
        if (id == 1009) {  
            //处理杀1009怪的成就逻辑  
        } else if (id == 1010) {  
            //处理杀1010怪的成就逻辑  
        }  
    }  
}  
  
main () {  
    //killMonster是杀死一个怪物，返回杀死怪物的id  
    int id = killMonster();  
    //发送消息，杀死怪物，并将杀死的怪物id一起发送  
    Notification.getInstance().sendMsg(EVENT_KILL_MONSTER, id);  
}
```
然后在写一个成就管理类来管理各种成就观察者，这样各个成就直接也可以解耦。
## 其他问题
- 引用销毁问题：这个问题容易造成内存泄漏，就是在这个观察者不再使用时，一定记得将其remove，否则这个观察者一直在引用着，不会被释放。
- 同步异步问题：sendMsg这个函数中是在主线程按加入顺序进行发送的，在特殊情况下根据需要可以使用多线程来实现。
- 其他应用：观察者模式在MVC这种结构下也经常使用，control来处理逻辑，通过观察者来相应UI事件。
观察者模式的优点就是可以做到完全的解耦；缺点就是使用不当会让程序难以维护和调试。