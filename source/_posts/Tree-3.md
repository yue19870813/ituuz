---
title: xtJS初级教程之ExtJS Tree(三)
date: 2016-11-27 22:33:05
tags: ExtJS
categories: javascript
---
前两次我介绍了静态树和根据数据库加载的数据生成的树，今天我就把ExtJS Tree这剩的一些主要的东西说一说，剩下的主要就是：树的事件处理、可编辑的树和可拖拽的树，最后再实现一下异步加载的树。<!--more-->
## 树的事件处理
树的事件主要有：1、展开节点事件。2、折叠节点事件。3、单击节点事件。4、双击节点事件。下面我们以一个静态树来测试树的主要事件。
``` javascript
//普通的静态树  
var tree = new Ext.tree.TreePanel({  
 el:'tree-div'  
});  
//创建根节点  
var root = new Ext.tree.TreeNode({text:'AllShengFen'});  
//创建父节点  
var shengfen1 = new Ext.tree.TreeNode({text:'HeiLongJiang'});  
var shengfen2 = new Ext.tree.TreeNode({text:'LiaoNing'});  
var city1 = new Ext.tree.TreeNode({text:'harbin'});  
var city2 = new Ext.tree.TreeNode({text:'daqiang'});  
var city3 = new Ext.tree.TreeNode({text:'shenyang'});  
var city4 = new Ext.tree.TreeNode({text:'dalian'});  
//将子节点添加到父节点中  
shengfen1.appendChild(city1);  
shengfen1.appendChild(city2);  
shengfen2.appendChild(city3);  
shengfen2.appendChild(city4);  
//将子节点添加到父节点中  
root.appendChild(shengfen1);  
root.appendChild(shengfen2);  
//将root节点设置为根节点  
tree.setRootNode(root);  
//使这棵树显示在div标签中  
tree.render();  
//展开节点事件  
tree.on("expandnode",function(node){  
 alert("["+node.text+"]open");  
});  
//折叠节点事件  
tree.on("collapsenode",function(node){  
 alert("["+node.text+"]close");  
});  
//单击节点事件  
tree.on("click",function(node){  
 alert("["+node.text+"]click");  
});  
//双击节点事件  
tree.on("dblclick",function(node){  
 alert("["+node.text+"]double click");  
}); 
```
由于单击和双击有冲突，所以我们在测试的时候可以先注释掉一个，单独测试。
## 可编辑的树
有了前面我们ExtJS的基础我就直接上代码了。简单的功能我都在代码的注释里写了。下面的例子中都会用到TreeNodeServlet.Java这个类所以我把这个代码也贴出来。
``` java
public class TreeNodeServlet extends HttpServlet {  
    private static final long serialVersionUID = 1L;  
         
    /** 
     * @see HttpServlet#HttpServlet() 
     */  
    public TreeNodeServlet() {  
        super();  
        // TODO Auto-generated constructor stub  
    }  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response)  
            throws ServletException, IOException {  
      
        response.setContentType("text/html;charset=utf-8");  
        PrintWriter out = response.getWriter();  
        String node = request.getParameter("node");  
        String json = "";  
        if("0".equals(node)){  
              
            json += "[{id:1,text:'节点1'},{id:2,text:'节点2'}]";  
        }  
        else if("1".equals(node)){  
              
            json += "[{id:11,text:'节点11',leaf:true},{id:12,text:'节点12',leaf:true}]";  
              
        }  
        else if("2".equals(node)){  
              
            json += "[{id:21,text:'节点21'},{id:22,text:'节点22',leaf:true}]";  
              
        }  
        else if("21".equals(node)){  
              
            json += "[{id:211,text:'节点211',leaf:true},{id:212,text:'节点212',leaf:true}]";  
              
        }  
          
        out.write(json);  
          
    }  
} 
```
前端代码
``` javascript
var tree = new Ext.tree.TreePanel({  
 el:'tree-div',  
 loader: new Ext.tree.TreeLoader({dataUrl:'../TreeNodeServlet'})  
});  
var root = new Ext.tree.AsyncTreeNode({  
 id:'0',text: 'harbin'  
});  
tree.setRootNode(root);  
tree.render();  
//设置该树可编辑  
var treeEditor = new Ext.tree.TreeEditor(tree,{allowBlank:false});  
//设置只有叶子节点可编辑  
treeEditor.on("beforestartedit",function(treeEditor){  
 return treeEditor.editNode.isLeaf();  
});  
tree.on("click",function(node){  
   
 alert("The name is:"+node.text+"***"+node.id);  
   
});  
//此处将编辑的值输出  
treeEditor.on("complete",function(treeEditor,newValue){  
 alert("The name had been modified,the result is:"+newValue);  
});  
```
## 可拖拽的树
``` javascript
var tree = new Ext.tree.TreePanel({  
 el:'tree-div',  
//设置可拖拽  
 enableDD:true,  
 loader: new Ext.tree.TreeLoader({dataUrl:'../TreeNodeServlet'})  
});  
var root = new Ext.tree.AsyncTreeNode({  
 id:'0',  
    text: 'harbin'  
});  
tree.setRootNode(root);  
tree.render();  
//设置叶子节点可追加元素  
tree.on("nodedragover",function(e){  
 var node = e.target;  
 if(node.leaf)  
  node.leaf = false;  
 return true;  
}); 
```
## 异步加载树
异步加载树是非常常用的，因为我们页面上生成的树往往都是从数据库中查出来的数据，有时候我们的数据量很大，如果每次都全加载那么很浪费资源，所以我们就用到了异步加载的树。所谓异步加载的树就是：我们点击哪个节点，就只加载这个节点下的元素，并不查询其他元素。但数据量大，并且树的层次特别多的时候就非常好用了，下面我们就实现一个异步加载的树。我将所有代码都粘出来，方便大家直接运行测试。
- 准备数据库的数据。这个我们就用在第二节给的数据就可以了。
- vo也就是JavaBean的写法，我们也在第二节将过了，下面是代码。
``` java
public class Tree {  
    private int id;  
    private String text;  
    private boolean leaf;  
    public Tree(){  
          
    }  
    public Tree(int id,String text,boolean leaf){  
          
        this.id = id;  
          
        this.text = text;  
          
        this.leaf = leaf;  
          
    }  
      
    public int getId() {  
        return id;  
    }  
    public void setId(int id) {  
        this.id = id;  
    }  
    public String getText() {  
        return text;  
    }  
    public void setText(String text) {  
        this.text = text;  
    }  
    public boolean isLeaf() {  
        return leaf;  
    }  
    public void setLeaf(boolean leaf) {  
        this.leaf = leaf;  
    }  
}  
```
- 接着是查询的DAO，其实也和第二节的差不多，只不过第二节用的是查询所有，而这里我们只是查询点击的节点下的数据，所以我们就根据传递过来的id进行查询部分数据进行显示。
``` java
public class DAO {  
    Connection conn;  
      
    PreparedStatement pstate;  
      
    ResultSet rs;  
      
    public List findAll(int id){  
          
        List list = new ArrayList();  
          
        String sql = " select deptid,deptname from dept where higherDept = "+id ;  
          
        try{  
              
            conn = DBConn.getConn();  
              
            pstate = conn.prepareStatement(sql);  
              
            rs = pstate.executeQuery();  
              
            while(rs.next()){  
                  
                Tree vo = new Tree();  
                  
                vo.setId(rs.getInt("deptid"));  
                vo.setText(rs.getString("deptname"));  
                vo.setLeaf(false);  
                list.add(vo);  
            }  
              
              
        }catch(Exception e){  
              
            e.printStackTrace();  
              
        }finally{  
              
            DBConn.closeRs(rs);  
              
            DBConn.closePreState(pstate);  
              
            DBConn.closeConn(conn);  
              
        }  
          
        return list;  
    }  
} 
```
这里涉及到了另外一个工具类，就是连接数据库用的类，这个非常简单大家都应该没有问题。
- 然后是servlet，servlet里我们也简单的处理一下，就是接收一下页面传递过来的节点id，并根据这个id查询数据。
``` java

public class TreeServlet extends HttpServlet {  
    private static final long serialVersionUID = 1L;  
         
    /** 
     * @see HttpServlet#HttpServlet() 
     */  
    public TreeServlet() {  
        super();  
        // TODO Auto-generated constructor stub  
    }  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response)  
            throws ServletException, IOException {  
        response.setContentType("text/html;charset=utf-8");  
        PrintWriter out = response.getWriter();  
        String node = request.getParameter("pid");  
          
        int id = Integer.parseInt(node);  
          
        DAO dao = new DAO();  
        List list = new ArrayList();  
          
        list = dao.findAll(id);  
                response.getWriter().write(JSONTools.getJsonArray(list).toString());  
          
    }  
}  
```
- 下面就是页面js文件了。
``` javascript
var tree = new Ext.tree.TreePanel({  
 el:'tree-div',  
 loader: new   
//第一次访问的时候只加载根节点下的数据  
Ext.tree.TreeLoader({dataUrl:'../TreeServlet?pid=23010000'})  
});  
var root = new Ext.tree.AsyncTreeNode({  
    text: 'harbin',  
    draggable:false,  
    id:'23010000'  
});  
tree.setRootNode(root);  
tree.render();  
//根据点击的节点加载该节点下的数据,点击哪个节点就加载哪个节点的数据  
tree.on('beforeload',function(node){     
    if(node.id != '23010000'){    
        tree.loader.dataUrl = '../TreeServlet?pid='+node.id;           
    }      
}); 
```
这样整个异步加载的树也就完成了。完成了异步加载的树，那么ExtJS Tree这块的主要内容也就差不多了，还有一些细节大家研究研究就可以了。如果哪有错误或不足希望指出。
