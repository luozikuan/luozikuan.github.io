---
layout: post
category: coding
title: Qt 与 Html 合作编程
tagline: 
tags: [qt5]
---
在界面编程方面，html可以快速实现想要的界面效果。Qt虽然有强大的qml，但是在有些场合，这需要一个过渡期。以前有开发者会将Qt不好做的界面用Html实现，然后使用Qt和Js实现数据交换，以完全整个软件功能。这里将讨论一下Qt和Html/Js的合作编程，主要内容是如何调用对方的函数。

<!--more-->

以下示例功能非常简单，仅实现了登录功能。使用html做页面，程序启动时，由QT把上次登录的用户名密码传递给html，用户重新输入用户名和密码后，点击登录，会调用js获取到用户名和密码，然后将数据传递给QT，QT判断该用户名和密码是否匹配。

## QWebView入门
Qt中有完整的webkit工具，其中包含QWebView，使用它使可以在软件中显示并操控html文件。使用QWebView需要在.pro工程文件中加入`QT += webkitwidgets`，然后`#include <QWebView>`即可。

### 准备工作
需要先准备好一个html文件，然后在Qt中使用以下代码加载此html文件：

~~~
    ui->webView->setUrl(QUrl("path/to/html.html"));
~~~

然后设置html的window名字，window名字是指在html中调用Qt时使用的一系列函数的前缀，稍后会详细说到。此处我全部设置为`xxx`。

~~~
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    
    ui->webView->setAcceptDrops(false);
    QString loginFile = "path/to/html.html";
    ui->webView->setUrl(QUrl(QString("file:///%1").arg(loginFile)));
    connect(ui->webView->page()->mainFrame(), SIGNAL(javaScriptWindowObjectCleared()),
            this, SLOT(addToJavascript()));
}

void Widget::addToJavascript() // slot
{
    ui->webView->page()->mainFrame()->addToJavaScriptWindowObject("xxx", this);
}            
~~~

## Js中怎么调用Qt函数

### 简单示例

如果需要在网页按钮按下时将文本框的数据传输给Qt程序，就需要在点击按钮时调用Qt的函数。

~~~html
<!--html文件局部-->
<input type="text" placeholder="username" class="login_input" id="username"/>
<input type="password" placeholder="password" class="login_input" id="password"/>
<input type="submit" class="login_submit">
~~~

~~~js
// js文件局部
$(".login_submit").click(function () {
	var username = $("#username").val();
	var password = $("#password").val();
	alert(username + password);
	window.xxx.submit(username, password);
});
~~~

`window.xxx.submit(username, password);`表示调用Qt中的submit槽函数，`username`和`password`为QString类型的传入参数。

~~~
// qt文件局部
void Widget::submit(QString username, QString password)
{
    if ("luozikuan" == username && "123456" == password) {
        qDebug() << tr("login success");
    } else {
        qDebug() << tr("wrong username or password");
    }
}
~~~

### 调用时的参数类型说明
因为js是动态类型的语言，其变量没有确定的类型，所以QT统一将JS传过来的参数当做QString类型。
例如：`window.xxx.func1(1)`中的1传递到QT之后变成`QString`类型的`"1"`。

`window.xxx.func2(function(){alert('aaa');})`的匿名函数参数传递到QT中变成`QString`类型的`"function(){alert('aaa');}"`。

## Qt中怎么调用Js函数
假如在这个例子中，我们需要让登录框在弹出时填入上次记住的username和password，那么就需要在Qt中取出这两个值，然后再调用Js的函数把这两个值放到相应的输入框中。

~~~
// qt文件局部
void Widget::onDocReady()
{
    QString username = getRememberedUsr();
    QString password = getRememberedPwd();
    QString js = QString("fillUsrAndPwd('%1','%2');").arg(username).arg(password);
    ui->webView->page()->mainFrame()->evaluateJavaScript(js);
}
~~~

~~~javascript
// js文件局部
function fillUsrAndPwd (username, password) {
	$("#username").val(username);
	$("#password").val(password);
}
window.xxx.onDocReady();
~~~

Js先调用Qt的`onDocReady`槽函数，`onDocReady`获取之前记录的用户名和密码，然后调用Js的`fillUsrAndPwd`函数将数据显示到界面上。

这么写好像有点复杂，为什么不直接在`Widget`的构造函数里调用`onDocReady`呢？

因为直接调用会涉及到Js是否可用的问题。

### 调用Js时Js是否已加载完
如果程序在`setUrl`之后马上调用js中的函数，有可能出现js函数还没加载完的情况，如果还没加载完，调用就没有任何效果，所以通常的做法是html写一个qt槽函数在js加载完成后调用，这样qt就可以感知到html/js已加载完，可以放心地调用js函数。

## 一些问题

### 图片浏览器？

使用QWebView之后，QWebView相当是一个浏览器，这时往里面拖拽一个文件时，它会试图打开这个文件。但这个逻辑并不是我们想要的。

解决方法也很简单，在QWebView初始化后，使用QWebView的`setAcceptDrops(false);`方法可以禁用QWebView的文件拖拽功能。

[完整代码下载]({{site.attachments_path}}/qwebview/login.zip)