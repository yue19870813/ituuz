---
title: ExtJS初级教程之ExtJS Grid(一)
date: 2016-11-27 22:45:35
tags: ExtJS
categories: javascript
---
我们在很多网站都能看到表格的影子，而ExtJS Grid在页面表现上又非常强大，下面我们就创建我们的第一个表格。<!--more-->
在ExtJS中我们是通过Ext.grid.ColumnModel这个类来描述列属性的。下面我们建立一个ColumnModel对象。
``` javascript
var cm = new Ext.grid.ColumnModel([  
 {header:'产品编号',dataIndex:'product_id'},  
 {header:'产品名称',dataIndex:'product_name'},  
 {header:'产品价格',dataIndex:'product_price'}  
]);
```
接下来我们定义表格中要显示的数据。
``` javascript
var data = [  
    ['01','电脑','4800'],  
    ['02','手机','2100'],  
    ['03','相机','1800'],  
    ['04','衣服','220']  
];  
```
数据中的列是和定义表格列属性时是相对应的。下面我们要创建一个Store对象来讲data装载到stroe对象中。
``` javascript
var store = new Ext.data.Store({  
 proxy:new Ext.data.MemoryProxy(data),  
 reader:new Ext.data.ArrayReader({},[  
       {name:'product_id'},  
       {name:'product_name'},  
       {name:'product_price'}  
 ])  
});  
```
proxy属性是一个代理类对象，该对象可以封装一个静态数组。ArrayReader对象可以将数组解析，但是要使用mapping属性来进行映射。
``` javascript
var store = new Ext.data.Store({  
 proxy:new Ext.data.MemoryProxy(data),  
 reader:new Ext.data.ArrayReader({},[  
       {name:'product_id',mapping:0},  
       {name:'product_name',mapping:1},  
       {name:'product_price',mapping:2}  
 ])  
});  
store.load();  
```
接着就是创建一个GridPanel对象来装置表格并显示在页面上。
``` javascript
var gridPanel = new Ext.grid.GridPanel( {  
 autoHeigth : false,  
 renderTo : 'grid',  
 store : store,  
 cm : cm  
});  
```
这样，我们第一个静态表格就完成了。下面是完整的代码：
``` javascript

Ext.onReady(function() {  
 var cm = new Ext.grid.ColumnModel( [ {  
  header : 'id',  
  dataIndex : 'product_id'  
 }, {  
  header : 'name',  
  dataIndex : 'product_name'  
 }, {  
  header : 'price',  
  dataIndex : 'product_price'  
 } ]);  
 var data = [ [ '01', 'computer', '4800' ], [ '02', 'phone', '2100' ],  
   [ '03', 'ffff', '1800' ], [ '04', 'closes', '220' ] ];  
 var store = new Ext.data.Store( {  
  proxy : new Ext.data.MemoryProxy(data),  
  reader : new Ext.data.ArrayReader( {}, [ {  
   name : 'product_id',  
   mapping : 0  
  }, {  
   name : 'product_name',  
   mapping : 1  
  }, {  
   name : 'product_price',  
   mapping : 2  
  } ])  
 });  
 store.load();  
 var gridPanel = new Ext.grid.GridPanel( {  
  autoHeigth : false,  
  renderTo : 'grid',  
  store : store,  
  cm : cm  
 });  
});  
```
通过这个简单的代码例子我们也大概了解了创建一个Grid的基本流程了，下面我们简单介绍一下Grid的一些其他的简单属性。

阻止移动列和改变列的宽度：
``` javascript
var gridPanel = new Ext.grid.GridPanel( {         
    autoHeigth : true,    
    renderTo : 'grid',    
    enableColumnMove:false,   
    enableColumnResize:false,     
    store : store,    
    cm : cm   
}); 
```
代码中enableColumnMove:false设置的是阻止列的移动，enableColumnResize:false设置的是让列的宽度禁止改变。

按列排序：
``` javascript
var cm = new Ext.grid.ColumnModel( [ {                
        header : 'id',        
        dataIndex : 'product_id'          
    }, {              
        header : 'name',          
        dataIndex : 'product_name',       
        sortable:true         
    }, {              
        header : 'price',         
        dataIndex : 'product_price'       
    } ]); 
```
在该列设置sortable：true之后，该列就拥有了排序的功能。

有很多时候我们需要在表格的每列前加一个复选框，用来标记选中的，进行操作，下面我们就加入复选框试试。首先，我们要创建复选框对象，这时我们要用到Ext.grid.CheckboxSelectionModel这个类。
``` javascript
var sm = new Ext.grid.CheckboxSelectionModel();           
var cm = new Ext.grid.ColumnModel( [              
              
   {              
    header : 'id',        
    dataIndex : 'product_id',         
    sortable:true         
}, {              
    header : 'name',          
    dataIndex : 'product_name',       
    sortable:true  
}, {      
    header : 'price',  
    dataIndex : 'product_price',  
    sortable:true  
} ]);  
```
创建完复选框对象后，我们还要将这个复选框对象添加到ColumnModel对象中去。接着还要将复选框对象添加到Panel上:
``` javascript
var gridPanel = new Ext.grid.GridPanel( {     
    autoHeigth : true,  
    renderTo : 'grid',  
    enableColumnMove:false,  
    enableColumnResize:false,  
    store : store,  
    cm : cm,  
    sm:sm  
});  
```
下面我们做一个非常频繁用到的功能：将复选框选中的行删除。首先要在页面添加一个删除按钮。
``` html
<input type = "button" value = "删除选中行" onclick = "deleteCheckbox()" style = "margin-top:20px"/>  
```
这个JavaScript函数就是页面按钮要调用的函数：
``` javascript
function deleteCheckbox(){                
     //从后往前扫描             
     for(var i = store.getCount();i>=0;i--){           
        if(mm.isSelected(i)){         
                //删除当前行  
                store.removeAt(i);  
            }  
     }        
     //重新加载表数据        
     gridPanel.reconfigure(store,cm);         
  }; 
```
这样一个主要功能都健全的Grid就完成了，今天我就写这些，下次我将继续ExtJS Grid的分页和编辑表格的研究。