---
title: 安装mysql出现no compatible servers were found
date: 2018-08-02 14:13:19
tags: [mysql]
categories: 教程
---


问题描述
---
今天在安装数据库的过程中，遇到错误提示：

	No compatible servers were found，You'll need to cancel this wizard and install one！
如下图所示：
![](/images/2018-8-1/mysql_install_errors.png)
精通于数据库的安装与卸载的我，竟然也有“湿鞋”的一天，懵逼一段时间之后，首先想到的就是以前的注册表信息没有删干净，嗯，一定是这样，果断去控制面板卸载mysql,找删除mysql注册表的相关博客，经过一段时间的忙碌之后，信誓旦旦的再次开始重装，然后，我被打脸了！！！
痛定思痛，努力排查，诶，这mysql server没装成功啊！如图所示：
![](/images/2018-8-1/mysql_server_error.png)
然后想到是不是需要装其他的软件，然后在再次卸载数据库之后我选择了装下面这一大坨东西，以前都是直接选择跳过的，满怀希望，默默等待......然而，他又又失败了，同样的错误。
![](/images/2018-8-1/mysql_install_ingnore.png)
最后根据软件缺失这个思路在网上找到了解决。
解决办法
---
安装失败的原因是需要升级一个插件，**Visual C++ 2013 and Visual C Redistributable Package**
**且必须是32位的Visual C++ Redistributable Packages for Visual Studio 2013！！！**
**注意是32位的，与电脑的系统类型无关，即32位，64位系统都要装32位的visual c++**
我选择从微软官网下载，下载网址： https://www.microsoft.com/zh-cn/download/details.aspx?id=40784
进入网址，点击下载，如下图：
![](/images/2018-8-1/vc_download.png)
选择vcredist_x86.exe—6.2 MB，下载并安装
![](/images/2018-8-1/vc_choose.png)
安装完成后，再次安装mysql数据库
![](/images/2018-8-1/otp_c_install.png)
![](/images/2018-8-1/mysqlserver_success.png)
![](/images/2018-8-1/mysql_success.png)
完成，问题解决！！！

问题解决方出自：
https://blog.csdn.net/q95548854/article/details/78780916