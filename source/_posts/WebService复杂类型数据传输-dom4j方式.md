---
title: WebService复杂类型数据传输-dom4j方式
date: 2016-11-27 19:39:57
tags: ［WebService,java］
categories: java
---
WebService在传递数据的时候只能传递字符串，当我们返回一些简单的字符串时我们可以直接返回，但是当我们想返回比如List，Map等复杂类型的数据时拼接字符串就是个很麻烦的工作，这时我们就用到了dom4j这个工具。

当我们从数据库中查询出很多个对象类型时，我们一般都存放在List中。像这种数据就很难用拼接字符串的形式进行传递。我的解决办法就是利用dom4j把它写成一个xml格式的字符串，然后在客户端再利用dom4j解释。
<!--more-->
假如我们从数据库中查询的是一个Student对象，该对象有name和agel两个属性，我们将大量的Student对象储存在List中。下面我们想利用dom4j把List数据变成如下格式的xml文件类型的字符串：  
``` xml
	<?xml version="1.0" encoding="UTF-8"?>  
    <root>  
        <student>  
            <name>张三</name>  
            <age>26</age>  
        </student>  
        <student>  
            <name>赵四</name>  
            <age>34</age>  
        </student>  
    </root> 
```
webservice端Java代码如下：  
``` java
import java.util.List;  
import org.dom4j.Document;  
import org.dom4j.DocumentHelper;  
import org.dom4j.Element;  
import org.dom4j.io.OutputFormat;  
import vo.UnitPO;  
import DAO.unitDao;  
public class Service02 {     
    public String method(){  
         
        Document document = DocumentHelper.createDocument();  
        Element root = document.addElement("root");  
         
        Element stuElement = null;  
        Element nameElement = null;  
        Element ageElement = null;  
         
        //第一个student节点  
        stuElement = root.addElement("student");  
        //添加两个子节点  
        nameElement = stuElement.addElement("name");  
        ageElement = stuElement.addElement("age");     
        //向两个子节点中添加文本内容  
        nameElement.addText("张三");  
        ageElement.addText("26");  
         
        //同理第二个student节点  
        stuElement = root.addElement("student");         
        nameElement = stuElement.addElement("name");  
        ageElement = stuElement.addElement("age");     
        nameElement.addText("赵四");  
        ageElement.addText("34");  
        try {  
            OutputFormat format = OutputFormat.createPrettyPrint();  
            // 设置XML文件的编码格式  
            format.setEncoding("UTF-8");  
            return document.asXML();  
        } catch (Exception e) {  
            System.out.println(e.getMessage());  
            return "error";  
        }  
    }  
} 
```
这样就将我们想传送的复杂数据变成了xml格式是字符串，下面在客户端就可以解释出我们想要的数据了。client端java代码如下：
``` java
import java.io.StringReader;  
import java.util.List;  
import org.dom4j.Document;  
import org.dom4j.Element;  
import org.dom4j.io.SAXReader;  
import webservice.Service02Proxy;  
public class Test {  
    public static void main(String[] args) {         
        try{  
            //创建代理类  
            Service02Proxy proxy = new Service02Proxy();  
            //接受传过来的XML字符串  
            String str = proxy.method();  
            //解释该字符串  
            StringReader read = new StringReader(str);  
            SAXReader reader = new SAXReader();  
            Document doc = reader.read(read);   
            Element root = doc.getRootElement();  
            //该节点由多个student构成，所以返回值是List类型  
            List list = root.elements("student");   
             
            for(int i = 0;i < list.size();i++){  
                 
                Element student = (Element)list.get(i);  
                Element name = student.element("name");  
                Element age = student.element("age");  
                 
                //打印得到的数据  
                System.out.println(name.getTextTrim()+":"+age.getTextTrim());  
                 
            }  
     
        }catch(Exception e){  
             
            e.printStackTrace();  
             
        }  
    }  
}
```
控制台输出为：
> **张三:26
赵四:34 **

每种编程语言可能接受时用的类型不同，但方法大同小异。代码如有问题，欢迎给予评价。