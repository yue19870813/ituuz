---
title: 设计模式在游戏开发中的应用之单例模式
date: 2016-12-01 23:07:14
tags: 单例模式
categories: 设计模式
---
# 单例模式
## 通俗的定义
是指在运行中只有一个实例对象存在。
## 结构图如下
![single](/images/design_3_single.png)
<!--more-->
## 游戏开发中的使用
游戏开发中单例模式的使用也是非常普遍的，比如在Cocos2d-x中的Director就是一个单例。比如游戏中的很多工具类都是做成单例或者静态类的方式来使用。单例还有一种写法，很少有人使用，然而这种写法在做游戏开发时却很好用。比如，我们在需要分享时，往往都需要每个平台都有单独的功能和实现方式，很多时候我们都这么写：
``` java
class ShareUtils {  
    private _instance = new ShareUtils();  
  
    private ShareUtils() {  
    }  
  
    public getInstance() {  
        return _instance;  
    }  
      
    public void shareToIOS () {  
        //todo iOS share  
    }  
      
    public void shareToAndroid () {  
        //todo android share  
    }  
}  
```
然后再根据不同平台调用不同的函数。其实我们完全可以写的再优雅一些：
``` java
class ShareUtils {  
    private ShareUtils _instance = null;  
  
    private ShareUtils() {  
    }  
  
    public getInstance() {  
        if (_instance == null) {  
            if (PLATFORM == IOS) {  
                _instance = new IOSShare()  
            } else if (PLATFORM == ANDROID) {  
                _instance = new AndroidShare()  
            }  
        }  
        return _instance;  
    }  
      
    public void share();  
}  
  
class IOSShare extends ShareUtils {  
    public void share() {  
        //todo ios share  
    }  
}  
  
class AndroidShare extends ShareUtils {  
    public void share() {  
        //todo android share  
    }  
}  
```
其实这里我们更重要的是要讨论一下单例模式的问题。
## 单例模式的问题
最大问题之一就是它本身是一个全局变量。全局变量会让人很难阅读和理解，当我们去查找一个别人写的代码中的bug时，如果这里没有使用全局变量的话，我们只要理解这个函数体内的代码和传递的参数就可以了。然而当这里充斥着全局变量的时候，性质就不一样了，你要全局搜索这个全局变量都在哪里引用了，做了什么修改，为什么这么修改，理解和修改的代价就会变得特别大。同时全局变量还增加了代码的耦合性，这也是个问题。在很多项目中我们都能看见这样的类：SoundManager,GameManager,DataUtils等等，各种各样的Manager和Utils，大多数时候它们很管用，但是当你要创建这么一个类的时候，你应该思考一下真的需要一个单例的类么？在《游戏编程模式》中有下面这个例子：
``` java
class Bullet  
{  
public:  
  int getX() const { return x_; }  
  int getY() const { return y_; }  
  
  void setX(int x) { x_ = x; }  
  void setY(int y) { y_ = y; }  
  
private:  
  int x_, y_;  
};  
  
class BulletManager  
{  
public:  
  Bullet* create(int x, int y)  
  {  
    Bullet* bullet = new Bullet();  
    bullet->setX(x);  
    bullet->setY(y);  
  
    return bullet;  
  }  
  
  bool isOnScreen(Bullet& bullet)  
  {  
    return bullet.getX() >= 0 &&  
           bullet.getX() < SCREEN_WIDTH &&  
           bullet.getY() >= 0 &&  
           bullet.getY() < SCREEN_HEIGHT;  
  }  
  
  void move(Bullet& bullet)  
  {  
    bullet.setX(bullet.getX() + 5);  
  }  
}; 
```
这里的BulletManager就是一个管理Bullet的单例类，看起来这里很合理，但是真的需要吗？答案是不需要：
``` java
class Bullet  
{  
public:  
  Bullet(int x, int y) : x_(x), y_(y) {}  
  
  bool isOnScreen()  
  {  
    return x_ >= 0 && x_ < SCREEN_WIDTH &&  
           y_ >= 0 && y_ < SCREEN_HEIGHT;  
  }  
  
  void move() { x_ += 5; }  
  
private:  
  int x_, y_;  
};  
```
总之，单例是很方便，有时也很管用，但是要慎用，如果项目中充斥着大量的单例，那么这个项目肯定是难以维护的。