---
layout: post
category: coding
title: Qt 与 Html 合作编程
tagline:
tags: [qt5]
---
在界面编程方面，html 可以快速实现想要的界面效果。Qt 虽然有强大的 qml，但是在有些场合，这需要一个过渡期。以前有开发者会将Qt不好做的界面用Html实现，然后使用Qt和Js实现数据交互，以完成整个软件功能。这里将讨论一下 Qt 和 Html/Js 的合作编程，主要内容是如何调用对方的函数。

<!--more-->

以下示例功能非常简单，仅实现了登录功能。使用 html 做界面，程序启动时，由 QT 把上次登录的用户名密码传递给 html，用户重新输入用户名和密码后，点击登录，js 会获取到用户名和密码，然后将数据传递给 QT，QT 判断该用户名和密码是否匹配。

## QWebView入门
Qt 提供了 webkit 模块，其中包含QWebView，使用它使可以在界面上显示 html 网页。使用 QWebView 需要在 .pro 工程文件中加入 `QT += webkitwidgets`，然后在 Qt 文件中`#include <QWebView>`即可。

### 准备工作
需要先准备好一个 html 文件，然后在 Qt 中使用以下代码加载此 html 文件：

~~~cpp
    ui->webView->setUrl(QUrl("path/to/html.html"));
~~~

然后需要将能被 js 交互的 QObject 对象注入到 js 中，此过程需要开发者协商一个通讯的通道，也就是后续在 js 中调用 Qt 时使用的一系列函数的前缀，稍后会详细说到。此处我全部设置为`xxx`。

~~~cpp
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

如果需要在网页按钮按下时将文本框的数据传输给 Qt 程序，就需要在点击按钮时调用 Qt 的函数。

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

~~~cpp
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
因为 js 和 qt 的差异性，交互传参只支持简单类型，例如 QString, int, bool等，js 里其它 QT 不支持的类型会被统一处理成 QString 类型。
例如：`window.xxx.func1(true)`中的`true`传递到 qt 之后变成`bool`类型的`true`。

`window.xxx.func2(function(){alert('aaa');})`的匿名函数参数传递到 qt 中变成`QString`类型的`"function(){alert('aaa');}"`。

## Qt中怎么调用Js函数
在本文例子中，如果我们需要让登录框在弹出时填入上次记住的 username 和 password，那么就需要在 qt 中取出这两个值，然后再调用 js 的函数把这两个值放到相应的输入框中。

~~~cpp
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
如果程序在`setUrl`之后马上调用 js 中的函数，有可能出现 js 还没加载完的情况，如果还没加载完，调用就没有任何效果，所以通常的做法是写一个 qt 槽函数在 js 加载完成后调用，这样 qt 就可以感知到 html/js 已加载完，可以放心地调用 js 函数。

## 一些问题

### 图片浏览器？

使用QWebView之后，QWebView相当是一个浏览器，这时往里面拖拽一个文件时，它会试图打开这个文件。但这个逻辑并不是我们想要的。

解决方法也很简单，在QWebView初始化后，使用QWebView的`setAcceptDrops(false);`方法可以禁用QWebView的文件拖拽功能。

[完整代码下载]({{site.attachments_path}}/qwebview/login.zip)
