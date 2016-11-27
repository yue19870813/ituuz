---
title: Java多线程实现生产者与消费者
date: 2016-11-27 20:32:32
tags: 多线程
categories: java
---
生产者与消费者问题是指创建一组“生产者”线程和一组“消费者”线程，让他们进行同步互斥的运行，生产者生产一个产品，消费者就消费掉一个产品，下面我就用Java的多线程来实现这个课题。<!--more-->本例子的ChuShi代表生产者，XiaoFei代表消费者。首先是生产者厨师的代码：
``` java
package thread;  
public class ChuShi extends Thread{  
      
    private GongNeng g;  
    public ChuShi(GongNeng g){    
        this.g = g;  
    }  
      
    public void run(){  
        for(;;){  
              
            g.lao();  
              
        }  
    }  
} 
```
然后是消费者的代码：  
``` java
package thread;  
public class XiaoFei extends Thread{  
      
    private GongNeng g;  
      
    public XiaoFei(GongNeng g){  
          
        this.g = g;  
          
    }  
      
    public void run(){  
        for(;;){  
                  
            g.chi();  
              
        }  
    }  
}
```
接着是封装了生产者的生产方法和消费者的消费方法的类，该类中的方法要使用synchronized关键字，就可以使run方法同步，也就是说，对于同一个Java类的对象实例，run方法同时只能被一个线程调用，并当前的run执行完后，才能被其他的线程调用。需要注意的是每个线程run()方法调用的synchronized修饰的方法必须是一个实例的方法才能保证同步，所以这个里将生产者和消费者的功能封装在一个实例中，在将他们的实例分别传给生产者和消费者，这样他们就能够同步了，下面是同步的方法代码：
``` java
package thread;  
public class GongNeng {  
    private Bing b;  
      
    public GongNeng(Bing b){      
        this.b = b;   
    }  
      
    public synchronized void lao(){  
          
        if(b.getBing()<10){  
              
            try {  
                Thread.sleep(1500);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
              
            System.out.println(Thread.currentThread()+  
                  
            ":厨师烙了一张饼，盘里还剩"+(b.getBing()+1)+"张饼");  
              
            b.setBing(b.getBing()+1);  
        }  
    }  
      
    public synchronized void chi(){  
        if(b.getBing()>0){  
              
            try {  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
            System.out.println(Thread.currentThread()+  
                  
            ":消费者吃了一张饼，盘里还剩"+(b.getBing()-1)+"张饼");  
              
            b.setBing(b.getBing()-1);  
              
        }  
    }  
} 
```
还有他们对一个消费品进行同步的操作，在这里我们用的是饼：
``` java
package thread;  
public class Bing {  
    private int bing = 10;  
    public int getBing() {  
        return bing;  
    }  
    public void setBing(int bing) {  
        this.bing = bing;  
    }  
      
}
```
下面是主入口方法进行测试：
``` java
package thread;  
public class Test {  
    public static void main(String[] args) {  
          
        //创建饼的实例  
        Bing b = new Bing();  
        //创建同步的吃饼和烙饼实例  
        GongNeng g = new GongNeng(b);  
        //创建线程两个消费者两个生产者  
        ChuShi c1 = new ChuShi(g);  
        ChuShi c2 = new ChuShi(g);  
        XiaoFei x1 = new XiaoFei(g);  
        XiaoFei x2 = new XiaoFei(g);  
          
        //启动线程  
        c1.start();  
        c2.start();  
        x1.start();  
        x2.start();  
    }  
} 
```
 这个例子就模拟了生产与消费的问题。
