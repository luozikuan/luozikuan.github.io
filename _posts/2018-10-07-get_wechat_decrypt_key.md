---
layout: post
category: computer
title: 解密 PC 版微信数据库
tagline: 使用 OllyDbg 获取解密密钥
tags: [计算机, 微信]
---

PC 版微信在聊天记录的搜索和过滤上操作性不够自由，如果能将保存在本地的微信聊天数据库解密出来，配合专业的数据库软件（或者自己编写相应的数据过滤程序），自由度将会大大提升。本文使用 OllyDbg 调试微信得到数据库的密钥。  
以下内容在 PC 版微信v2.6.4 上测试有效。

<!--more-->

## 步骤如下
1. 打开微信，但是不要登录
   ![]({{site.images_path}}/wechat_decrypt/01.png)

2. 打开 OllyDbg.exe，选择"File">"Attach"附加到进程
   ![]({{site.images_path}}/wechat_decrypt/02.png)

3. 在弹出的选择框中选择 WeChat（注意不要选成 WeChatWeb）
   ![]({{site.images_path}}/wechat_decrypt/03.png)

4. 在 OllyDbg 的菜单栏中选择 "View">"Executable modules"
   ![]({{site.images_path}}/wechat_decrypt/04.png)

5. 在弹出的窗口里找到 "WeChatWin.dll" 并双击
   ![]({{site.images_path}}/wechat_decrypt/05.png)

6. 新弹出的窗口被分成了四块，在左上区域点鼠标右键，选择 "Search for">"All referenced text strings"，选择之后需要稍等几秒钟
   ![]({{site.images_path}}/wechat_decrypt/06.png)

7. 在新弹出的窗口里点鼠标右键，选择 "Search for text"
   ![]({{site.images_path}}/wechat_decrypt/07.png)

8. 输入 "DBFactory::encryptDB"，并点击 OK 按钮
   ![]({{site.images_path}}/wechat_decrypt/08.png)

9. 窗口中相应的行会出现灰色高亮，鼠标双击此行将会跳转到相应的汇编代码
   ![]({{site.images_path}}/wechat_decrypt/09.png)

10. 在此行下面第七行有一行汇编代码是 "TEST EDX,EDX"，选中它，并设置断点（快捷键 F2），设置断点后此行前面会变成红色
    ![]({{site.images_path}}/wechat_decrypt/10.png)

11. 点击 OllyDbg 的工具栏上的运行按钮（快捷键 F9）
    ![]({{site.images_path}}/wechat_decrypt/11.png)

12. 回到微信界面，登录，由于 OllyDbg 设置了断点的缘故，微信会停在断点处，并不会登录上去
    ![]({{site.images_path}}/wechat_decrypt/12.png)

13. 回到 OllyDbg，会发现程序已经停在了断点处（行首是黑底红色），此时在右上区域的 EDX 行点鼠标右键，在弹出的菜单项里选择 "Follow in Dump"，左下区域就会出现相应的内存数据
    ![]({{site.images_path}}/wechat_decrypt/13.png)

14. 用鼠标框选前四行内容，然后鼠标右键，在弹出的菜单项里选择 "Binary">"Binary copy"
    ![]({{site.images_path}}/wechat_decrypt/14.png)

15. 粘贴出来的文本去掉空格之后就是密钥。

