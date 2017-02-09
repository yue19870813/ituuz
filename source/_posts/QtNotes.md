---
title: Qt学习笔记
date: 2017-02-09 20:07:29
tags: Qt
categories: Qt
---
前几天项目需要一个编辑器，现学现卖边查文档边用Qt写了个简单的编辑器供项目使用，编写过程中记录了一些笔记，都是些没有系统性的知识点。
- 在Qt中固定窗口大小
```
//将最大窗口与最小窗口设置大小相同
setMinimumSize(1280, 720);
setMaximumSize(1280, 720);
```
- QString转char*
```
QString name = "test";
QByteArray ba = name.toLatin1();
char *mm = ba.data();
qDebug(mm);
```
- 输入对话框
```
QString name = QInputDialog::getText(this, tr("title") ,tr("Please input"));
```
<!--more-->
- 提示框
```
QMessageBox::about(NULL, "Warning", "Filename cannot be empty!");
```
- C1083: Cannot open include file: 'QAxObject': No such file or directory
```
#include <ActiveQt/QAxWidget>
```
- 浏览目录
```
QString openFilename = QFileDialog::getExistingDirectory(
                this,
                tr("Open fish table"),
                "",
                QFileDialog::ShowDirsOnly|QFileDialog::DontResolveSymlinks);
```
- 浏览文件
```
QString openFilename = QFileDialog::getOpenFileName(
                this,
                tr("Open fish table"),
                "",
                tr("Config Files (*.xlsx)"));
```
- 创建按钮不显示
```
auto btn = new QPushButton(this);
btn->setText("OK");
btn->move(100, 100);
btn->show();    //一定要调用show()才会显示
```
- 设置鼠标样式
```
setCursor(QCursor(Qt::OpenHandCursor));
```
- 创建json串
```  
QJsonObject json;
json.insert("name", "test");
json.insert("version", "1.0.1");

QJsonDocument document;
document.setObject(json);
QByteArray byte_array = document.toJson(QJsonDocument::Compact);
QString json_str(byte_array);
```
- 解析json串
```
QJsonParseError error;
QJsonDocument jsonDocument = QJsonDocument::fromJson(json.toUtf8(),&error);
if (error.error == QJsonParseError::NoError) {
    if (jsonDocument.isObject()) {
        QVariantMap result = jsonDocument.toVariant().toMap();
        resPath = result["resPath"].toString();
        fishTable = result["fishTable"].toString();
    }
}
```
- 发布打包  
1.首先使用Qt Creator发布Release版本。  
2.将Release目录下的.exe文件单独copy到一个目录中。  
3.命令行进入该目录执行your Qt path\bin\windeployqt project.exe  
4.全选生成的依赖文件，打包成rar压缩文件project.rar。  
5.打开你压缩的project.rar压缩包，点击菜单栏的解压格式，高级自解压选项。  
6.设置----解压后运行里写入project.exe（写你要执行的文件）。  
7.模式----打钩解包到临时文件夹，安全模式选择全部隐藏。  
8.更新----更新方式，解压并更新文件；覆盖凡是，覆盖所有文件。  
9.文本和图标------可改可不改。  
10.然后确定就可以生成exe可执行文件了。  
