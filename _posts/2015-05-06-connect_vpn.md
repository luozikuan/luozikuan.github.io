---
layout: post
category: computer
title: 如何连接VPN
tags: [计算机, VPN]
---
## VPN的用途

对于一个公司来说，数据的安全性非常重要。如果公司的关键文件遭泄露的话，损失将会很大。但是很多情况下，公司会有出差任务，或者在多地设有部门，那么员工在出差或异地子部门之间联系时，如何让通讯数据更安全呢？在这种场景下使用 VPN 可以提高数据安全性。

<font color="red">多图预警</font>

<!--more-->

## VPN的工作原理
VPN 有客户端和服务器，一般来说，公司内部会搭建一个 VPN 服务器，当员工（客户端）出差在外时，使用 VPN 拨号连接，可以把自己的上网设备和公司构成一个局域网，这之间的数据会加密传输，即使员工所连接的网络中存在监听者，也很难解密这些数据。

## 如何连接VPN
VPN有很多种，常见的有 PPTP，IpSec，OpenVPN 等。以下用 PPTP 为例说明如何连接VPN。

### windows 8
1. 鼠标悬停在屏幕右上或者右下角，然后点击“搜索”；
	![]({{site.images_path}}/vpn/windows/01.jpg)
2. 在搜索框中输入“vpn”，在结果中选择“管理虚拟专用网络”；
 	![]({{site.images_path}}/vpn/windows/02.jpg)
3. 点击右边的“添加VPN连接”；
	![]({{site.images_path}}/vpn/windows/03.jpg)
4. 填完这一页的基本信息。
	1. “VPN提供商”选择“Microsoft”；
	2. “连接名称”随意；
	3. “服务器名称或地址”填VPN提供商给的地址，此处填的“11.22.33.44”仅为示例；
	4. “用户名”和“密码”填上自己的账号密码。
	5. 点击“保存”。

	![]({{site.images_path}}/vpn/windows/04.jpg)
5. 现在VPN标签下已经有了刚才添加的那条记录；
	![]({{site.images_path}}/vpn/windows/05.jpg)
6. 退出到桌面，点击右下角的网络图标，选中刚才添加的VPN，点击连接；
	![]({{site.images_path}}/vpn/windows/06.jpg)
7. 连接后可能会出现691的错误，如果没有出现，那此连接就已经可以用了；
	![]({{site.images_path}}/vpn/windows/07.jpg)
8. 如果上一步出现了错误691，则*右击*右下角的网络图标，选择“打开网络和共享中心”；
	![]({{site.images_path}}/vpn/windows/08.jpg)
9. 点击左边的“更改适配器设置”；
	![]({{site.images_path}}/vpn/windows/09.jpg)
10. 在刚才添加的VPN连接上*右击*，选择“属性”；
 	![]({{site.images_path}}/vpn/windows/10.jpg)
11. 点击上边的“安全”标签页，将“VPN协议类型”选为“PPTP”；
	![]({{site.images_path}}/vpn/windows/11.jpg)
12. “数据加密”选“最大强度……”；
	![]({{site.images_path}}/vpn/windows/12.jpg)
13. “身份验证”选择“允许使用这些协议”，然后勾选下方的“MS-CHAP v2”。改完后点“确定”；
	![]({{site.images_path}}/vpn/windows/13.jpg)
14. 再次执行第7步，此时应该可以连接上了。
	![]({{site.images_path}}/vpn/windows/14.jpg)

### ubuntu 14.04
1. 点击右上角的“网络连接” -> "VPN connections" -> "configure VPN"；
	![]({{site.images_path}}/vpn/linux/01.jpg)
2. 在弹出的页面右边点击"Add"；
	![]({{site.images_path}}/vpn/linux/02.jpg)
3. 选择添加的网络类型为"PPTP"；
	![]({{site.images_path}}/vpn/linux/03.jpg)
4. 填上正确的"gateway", "User name"和"Password"；
	![]({{site.images_path}}/vpn/linux/04.jpg)
5. 点击"Advanced"，将设置改到跟图片一模一样；
	![]({{site.images_path}}/vpn/linux/05.jpg)
6. 点击"Save"保存连接；
	![]({{site.images_path}}/vpn/linux/06.jpg)
7. 点击右上角的“网络连接” -> "VPN connections" -> 刚才的网络连接 即可连接；
	![]({{site.images_path}}/vpn/linux/07.jpg)
8. 连接成功后网络连接图标会出现一把锁。
	![]({{site.images_path}}/vpn/linux/08.jpg)

### Mac OS X 10.10
1. 点击右上角的放大镜打开spotlight，输入“网络”，在结果中找到“系统偏好设置”分组下的“网络”，点击进入；
	![]({{site.images_path}}/vpn/mac/01.jpg)
2. 点击左下角的“+”新建一个网络连接；
	![]({{site.images_path}}/vpn/mac/02.jpg)
3. 在弹出的窗口中，“接口”填“VPN”，“VPN类型”选“PPTP”，“服务器名称”随意填；
	![]({{site.images_path}}/vpn/mac/03.jpg)
4. 在弹出来的界面中填上“服务器地址”，“账户名称”，“加密”选“最大”；
	![]({{site.images_path}}/vpn/mac/04.jpg)
5. 点击“鉴定设置”并输入用户密码；
	![]({{site.images_path}}/vpn/mac/05.jpg)
6. 点击“高级”，勾选上“通过VPN连接发送所有流量”；
	![]({{site.images_path}}/vpn/mac/06.jpg)
7. 勾选上“在菜单栏中显示VPN状态”；
	![]({{site.images_path}}/vpn/mac/07.jpg)
8. 点击右上角的“VPN状态”图标并连接刚才添加的VPN；
	![]({{site.images_path}}/vpn/mac/08.jpg)
9. 成功连接后会出现计时。
	![]({{site.images_path}}/vpn/mac/09.jpg)
