---
title: ExtJS初级教程之ExtJS Tree(二)
date: 2016-11-27 22:07:37
tags: ExtJS
categories: javascript
---
上次我将ExtJS Tree的静态树和加载服务器文件生成的树跟大家阐述了一下，今天我把怎么根据从数据库中查出的数据生成树根大家讨论一下。<!--more-->

要想根据数据库生成树首先我们先要准备数据库数据，大概表的结构如下：
![SQL](/images/tree_2_sql.png)
sql语句我也为大家准备好了，如下：
``` sql
//oracle建表语句  
create table  dept   (  
    deptId              NUMBER(20)                      not null,  
    deptName            VARCHAR2(20),  
    higherDept          NUMBER(20),  
   constraint PK_DEPT primary key ( deptId )  
);  
//测试数据  
insert into dept values(23010000,'哈尔滨市公安局',0);  
insert into dept values(23011000,'国家安全保卫队',23010000);  
insert into dept values(23011001,'国家安全保支队政工科',23010000);  
insert into dept values(23011002,'国家安全保支队秘书科',23010000);  
insert into dept values(23011003,'国家安全保支队一大队',23011001);  
insert into dept values(23011004,'国家安全保支队二大队',23011001);  
insert into dept values(23011005,'国家安全保支队三大队',23011004);  
insert into dept values(23011006,'国家安全保支队四大队',23011004);  
insert into dept values(23011007,'国家安全保支队五大队',23010000);  
insert into dept values(23011008,'国家安全保支队六大队',23011002); 
```
当数据都准备完毕后，首先要写一个vo用来储存我们将要查询出来的数据。这个vo属性的名字有一定的要求，必须遵守以下的命名： private int id;这个是保存数据的主键的，也就是deptId，这个id是生成树结构的关键。然后是 private String text;这个是生成树时显示的内容。最后是private boolean leaf;这个属性也挺关键，它是用来决定该树的节点是否是叶子节点。先不多说，等我们把树写完就了解这些属性的功能了。  
下面是DAO的查询方法，这个非常简单的用JDBC实现一下就好：
``` java
public class DAO {  
 Connection conn;   
 PreparedStatement pstate;  
 ResultSet rs;  
 public List findAll(){  
   
  List list = new ArrayList();  
  String sql = " select deptid,deptname from dept " ;   
  try{  
     
   conn = DBConn.getConn();   
   pstate = conn.prepareStatement(sql);  
   rs = pstate.executeQuery();  
     
   while(rs.next()){    
    Tree vo = new Tree();  
    vo.setId(rs.getInt("deptid"));  
    vo.setText(rs.getString("deptname"));  
    vo.setLeaf(true);  
    list.add(vo);  
   }   
  }catch(Exception e){     
   e.printStackTrace();     
  }finally{  
    //释放资源  
  }  
  return list;  
 }  
}
```
接着我们要在servlet中将查询出来的List数据转换成json字符串，这样ExtJS才能解析。
``` java
protected void service(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {  
 //转码  
 response.setContentType("text/html;charset=utf-8");  
 PrintWriter out = response.getWriter();  
    
 DAO dao = new DAO();  
 List list = new ArrayList();  
 list = dao.findAll();  
 //转换成json并传递给ExtJS Tree  
 response.getWriter().write(JSONTools.getJsonArray(list).toString());   
}  
```
代码中的JSONTools就是将List转换成json字符串的一个工具类。最后就是关键的页面树的实现部分了：
``` javascript
var tree = new Ext.tree.TreePanel({  
 el:'tree-div',  
 loader: new Ext.tree.TreeLoader({dataUrl:'../TreeNodeServlet'})  
});  
var root = new Ext.tree.AsyncTreeNode({  
    text: 'harbin',  
    id:'23010000'  
});  
tree.setRootNode(root);  
tree.render();
```
这样将项目部署到服务器上运行就可以看到效果了。


