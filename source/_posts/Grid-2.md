---
title: ExtJS初级教程之ExtJS Grid(二)
date: 2016-11-27 22:45:40
tags: ExtJS
categories: javascript
---
很多时候我们表格里显示的数据是从后台查询出来的海量数据，那么海量的数据显示在表格里用户的体验肯定会很差，效率也会很低。ExtJS为了解决这个问题，就给我们提供了一个非常强大的分页组件。今天我们就研究一下ExtJS的分页技术。<!--more-->
## 为表格添加分页组件
要为表格添加分页组件，就要先创建Ext.PagingToolbar类的实例对象，然后将它添加到Panel中去。
``` javascript
gridPanel = new Ext.grid.GridPanel({  
      autoHeight: true,  
      renderTo: 'grid',   
      store: store,       
      cm: cm,   
      sm:mm,  
      bbar:new Ext.PagingToolbar({  
      pageSize:3,  
      store:store,  
      displayInfo:true,  
      displayMsg:'显示记录 {0} - {1} of {2}',  
      emptyMsg:"没有记录"  
      })  
 }); 
```
PagingToolbar类的属性描述：
- pageSize：每页显示的记录条数。
- displayInfo：是否显示记录信息。
- displayMsg：该属性值为true时，该属性才有效。{0}：表示当前显示的记录起始数。{1}：表示显示记录数的结尾数。{2}：表示总共多少记录。
- emptyMsg：当没有记录的时候显示的文本信息。  

## 从服务器端获得分页数据
如果使用静态数据，Grid每次都会将所有数据显示。所以这里我们就编写一个Servlet来生成动态数据生成Grid。Servlet将接受两个请求数据start和limit。Start表示当前页显示记录的起始位置，limit表示每页显示的记录数。
``` java
public class GridServlet extends HttpServlet {  
 private static final long serialVersionUID = 1L;      
 protected void service(HttpServletRequest request, HttpServletResponse response)  
   throws ServletException, IOException {  
  response.setContentType("text/html;charset=gbk");  
  PrintWriter out = response.getWriter();  
  String start = request.getParameter("start");  
  String limit = request.getParameter("limit");  
  int index = Integer.parseInt(start);  
  int pageSize = Integer.parseInt(limit);  
  int total = 100000;  
  String jsonStr = "{total:" + total + ",root:[";  
  for (int i = index; i < pageSize + index; i++)  
  {  
   int productIndex = i + 1;  
   jsonStr += "{product_id:" + productIndex + ",product_name:'产品"  
     + productIndex + "',product_price:'价格" + productIndex  
     + "'}";  
   if (i != pageSize + index - 1)  
    jsonStr += ",";  
  }  
  jsonStr += "]}";  
  out.println(jsonStr);  
 }  
} 
```
要通过服务器来接收数据生成表格，就必须使用HttpProxy来制定URL。还有，服务器传递过来的数据是Json格式的，就不能使用ArrayReader了，需要使用的是JsonReader。
``` javascript
Ext.onReady(function(){  
 var mm = new Ext.grid.CheckboxSelectionModel();  
    var cm = new Ext.grid.ColumnModel([  
        new Ext.grid.RowNumberer(),  
        mm,  
        {header:'产品编号',dataIndex:'product_id'},  
        {header:'产品名称',dataIndex:'product_name'},  
        {header:'产品价格',dataIndex:'product_price'}  
    ]);    
 var store = new Ext.data.Store({  
        proxy: new Ext.data.HttpProxy({url:'../GridServlet'}),  
        reader: new Ext.data.JsonReader({  
         totalProperty:'total',  
         root:'root'  
        },[  
           {name:'product_id'},  
           {name:'product_name'},  
           {name:'product_price'}  
        ])   
    });  
store.load({params:{start:0,limit:10}});  
    var gridPanel = new Ext.grid.GridPanel({  
        autoHeight: true,  
        renderTo: 'grid',          
        store: store,  
        cm: cm,  
        sm:mm,  
        bbar:new Ext.PagingToolbar({  
         pageSize:10,  
         store:store,  
         displayInfo:true,  
         displayMsg:'显示记录 {0} - {1} of {2}',  
         emptyMsg:"没有记录"  
        })  
    });   
}); 
```
这样一个简单的ExtJS Grid分页就实现了。要想让用户体验更高就应该让用户可以在自己刚刚查询出来的数据表格上进行数据编辑，可以进行增删改查。可编辑的表格就在在想要能编辑的列里注册一个TextField组件，这个组件需要使用Ext.grid.GridEditor类来封装。  
## 第一个可编辑表格
``` javascript
cm = new Ext.grid.ColumnModel([  
new Ext.grid.RowNumberer(),  
 {header:'产品编号',dataIndex:'product_id'},  
 {header:'产品名称',dataIndex:'product_name',editor:new Ext.grid.GridEditor(new Ext.form.TextField({allowBlank:false}))},  
 {header:'产品价格',dataIndex:'product_price',editor:new Ext.grid.GridEditor(new Ext.form.TextField({allowBlank:false}))}  
]); 
```
使用了可编辑的对象就需要建立一个Ext.grid.EditorGridPanel来代替Ext.grid.GridPanel。
``` javascript
gridPanel = new Ext.grid.EditorGridPanel({  
  autoHeight: true,  
  renderTo: 'grid',      
  store: store,  
  cm: cm,  
});  
```
这时默认的是双击编辑，如果想设置成单击编辑的话就需要做如下改动。
``` javascript

gridPanel = new Ext.grid.EditorGridPanel({  
  autoHeight: true,  
  renderTo: 'grid',          
  store: store,  
  //值为1的时候就是单击，2就是双击。  
  clicksToEdit:1,  
  cm: cm,  
  sm:mm  
});
```
## 向表格中添加新行和删除一行
添加新的一行是使用Store类的insert方法插入一个新的Ext.data.Record对象，删除用的是remove方法,为了使插入的新行所有列都能编辑，我们要把所有列都设为可编辑。
``` javascript
cm = new Ext.grid.ColumnModel([  
  new Ext.grid.RowNumberer(),  
  mm,  
  {header:'产品编号',dataIndex:'product_id',editor:new Ext.grid.GridEditor(new Ext.form.TextField({allowBlank:false}))},  
  {header:'产品名称',dataIndex:'product_name',editor:new Ext.grid.GridEditor(new Ext.form.TextField({allowBlank:false}))},  
  {header:'产品价格',dataIndex:'product_price',editor:new Ext.grid.GridEditor(new Ext.form.TextField({allowBlank:false}))}  
]);
```
将所想要编辑的列添加Ext.grid.GridEditor对象就能是该列可编辑,接着我们要在表格组件的上方添加两个按键，分别是增加和删除。代码如下：
``` javascript
var gridPanel = new Ext.grid.EditorGridPanel({    
       autoHeight: true,      
       renderTo: 'grid',      
       store: store,      
       cm: cm,    
       tbar: new Ext.Toolbar(['-', {      
           text: '添加一行',      
           handler: function(){   
               var record = new Ext.data.Record({     
                product_id:'',  
                product_name:'',  
                product_price:''  
               });  
               gridPanel.stopEditing();                  
               store.insert(store.getCount(),record);  
               gridPanel.startEditing(store.getCount()-1,0);  
           }  
       }, '-', {  
           text: '删除一行',  
           handler: function(){  
               Ext.Msg.confirm('信息', '是否删除当前记录？', function(btn){  
                   if (btn == 'yes') {  
                       var sm = gridPanel.getSelectionModel();  
                       var cell = sm.getSelectedCell();  
                       var record = store.getAt(cell[0]);  
                       store.remove(record);  
                   }  
               });    
           }      
       }, '-'])   
   });  
```
## 保存数据
论是添加或是修改了表格中的数据后，一般我们都要将结果提交到服务器，在ExtJS中Slice类就是干这个用的。
``` javascript
{  
           text: '保存',    
           handler: function(){   
               var m = store.modified.slice(0);   
               var data = [];     
               Ext.each(m, function(item) {   
                   data.push(item.data);      
               });    
               alert(Ext.encode(data));   
               Ext.lib.Ajax.request(      
                   'POST',    
                   'SaveData',    
                   {success: function(response){      
                       Ext.Msg.alert('信息', response.responseText, function(){     
                           store.reload();    
                       });    
                   }},    
                   'row=' + encodeURIComponent(Ext.encode(data))      
               );     
           }      
}  
```
## 对记录进行分组  
此外ExtJS Grid还可以对数据进行分组，要建立一个按列分组的表格就要使用Ext.data.GroupingStore这个类。
``` javascript

var store = new Ext.data.GroupingStore({  
    reader: reader,  
    data: data,  
    groupField: 'sex',  
    sortInfo: {field: 'name', direction: "ASC"}  
});  
var grid = new Ext.grid.GridPanel({  
    autoHeight: true,  
    store: store,  
    columns: columns,  
    view: new Ext.grid.GroupingView(),  
    renderTo: 'grid'  
});  
```
ExtJS各种各样的组件和控件有很多，做的都很漂亮，还可以自己定制样式，使用继承来拓展ExtJS的功能，ExtJS的介绍就到这了，更多的功能我今后还会继续研究的。

