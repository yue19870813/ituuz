---
title: ExtJS初级教程之ExtJS Tree(一)
date: 2016-11-27 20:51:18
tags: ExtJS
categories: javascript
---
ExtJS是一款基于Ajax的web客户端框架，有着更加和漂亮友好的界面，今天起我就开始学习ExtJS了，下面我把我学习的过程与大家分享。<!--more-->  

我先学习我最近用到ExtJS Tree。下面我们开始写我们的第一个静态树。  
1、首先导入ExtJS提供的js和css文件。
``` javascript
  <link rel="stylesheet" type="text/css" href="../script/resources/css/ext-all.css" />
  <script type="text/JavaScript" src="../script/adapter/ext/ext-base.js"></script>
  <script type="text/javascript" src="../script/ext-all.js"></script>
  <script type="text/javascript" src="../script/locale/ext-lang-zh_CN.js"></script>
```
注意导入顺序，这些文件都可以在extjs的源码包里找到。  
2、首先我们先要创建一个空的树面板，此时页面上是神马都显示不出来的。
``` javascript
//创建Tree面板  
var tree = new Ext.tree.TreePanel({  
 //指定添加到的div  
 el:'tree-div',  
}); 
```
页面上应该有div的标签，因为我们写的树即将安插到这里。
``` javascript
<div id = "tree-div" ></div>
```
3、下面我们先创建一个根节点显示在页面上
``` javascript
//创建Tree面板  
var tree = new Ext.tree.TreePanel({  
 //指定添加到的div  
 el:'tree-div',  
});  
var root = Ext.tree.TreeNode({text:'跟节点'});  
tree.setRootNode(root);  
tree.render();  
```
这样页面上就不会是一个空树了，应该显示的是一颗带有一个根节点的树。  
4、最后我们按照上面的思路创建一个有层次的静态树。
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
```
这样一棵完整的树就应该显示在页面上了。  

但是我们大多数用的都不是写死的静态树而是要根据不同数据生成不同的树，接下来我们就看看怎么读取服务器端的文件来生成ExtJS Tree。  
1、创建一个data1.txt文件，内容是一个json字符串。
``` javascript
[  
 {text:'非叶子节点'},  
 {text:'叶子节点',leaf:true}  
] 
```
2、TreeLoader 导入数据的URL。我们需要一个类能够帮我们找到要加载的文件，这个类就是TreeLoader。
``` javascript
//创建Tree面板  
var tree = new Ext.tree.TreePanel({  
 //指定添加到的div  
 el:'tree-div',  
 //导入数据的URL  
 loader:new Ext.tree.TreeLoader({dataUrl:'../js/data1.txt'})  
}); 
```
3、但此时我们不能够再用Ext.tree.TreeNode来创建根节点了，用它创建根节点的话在页面上我们就只能看见跟节点。这是因为它不支持ajax，不能加载data.txt的数据，所以此时我们就用到了AsyncTreeNode了。
``` javascript
//创建根节点，使用AsyncTreeNode能够加载外部数据  
var root = new Ext.tree.AsyncTreeNode({text:'gen'});  
//设置root为根节点  
tree.setRootNode(root);  
//将树显示在div标签上  
tree.render();  
//第一个true是自动展开所有字节点，第二个true是以动画效果展开树  
root.expand(true,true); 
```
但此时我们发现，页面上的树开始无限的展开了。当我们将root.expand(true,true)删掉后，打开页面是不自动展开了，但是当我们点击展开的时候还是会无限的展开下去，这是因为AsyncTreeNode能够重新加载它下面的非叶子节点，我们加载的data1.txt里面的数据有一个节点是非叶子节点，所以AsyncTreeNode会再次的把它当做根节点，重新加载数据。所以我们应该给它一个所有节点的最后都是叶子节点的数据文件data2.txt。
``` javascript
[  
  {text:'computer',children:[  
    {text:'<a href="http://lib.csdn.net/base/17" class='replace_word' title="Java EE知识库" target='_blank' style='color:#df3434; font-weight:bold;'>Java</a>',leaf:true},  
    {text:'c++',children:[  
      {text:'thinking',leaf:true},  
      {text:'yyyyy',leaf:true}  
    ]},  
  {text:'asp',leaf:true}   
  ]},  
  {text:'jsp',children:[  
    {text:'dadf',leaf:true},  
    {text:'ddddddd',leaf:true}  
  ]}  
]  
```
此时我们将加载的数据源改为data2.txt。
``` javascript
//创建Tree面板  
var tree = new Ext.tree.TreePanel({  
 //指定添加到的div  
 el:'tree-div',  
 //导入数据的URL  
 loader:new Ext.tree.TreeLoader({dataUrl:'../js/data1.txt'})  
});
```
这时我们生成的树就不是无限展开的了。  
今天就这些，下次我们再研究从servlet中获得Json数据来生成ExtJS树。
