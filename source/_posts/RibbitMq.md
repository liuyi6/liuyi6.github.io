---
title: RabbitMQ安装教程
date: 2018-08-01 14:13:19
tags: [RabbitMQ]
categories: 教程
---
最近几天在学习Spring Cloud，在学习Spring Cloud Config配置刷新使用Spring Cloud Bus时，其中用到消息代理组件RabbitMQ,在安装RabbitMQ的过程查了很多资料，因此在这里将安装过程记录下来。
安装Erlang
----
由于RabbitMQ服务端代码是使用并发式语言erlang编写的，所以首先要安装erlang环境。
**Erlang下载**

下载地址是：http://www.erlang.org/downloads
我的测试环境是windows所以下载的是[OTP 20.2 Windows 64-bit Binary File](http://erlang.org/download/otp_win64_20.2.exe)
![](/images/2018-8-1/otp_download.png)
**Erlang安装**

下载完成后进行安装，可以按照习惯更改安装目录，在此过程中可能遇到安装Visual c++ 的情况不用管，一直点同意，等待安装完成。如下图所示：
![](/images/2018-8-1/otp_start.png)
![](/images/2018-8-1/otp_c_install.png)
**Erlang配置环境变量**

Erlang环境变量配置如下图所示：
![](/images/2018-8-1/erlang_home.png)
![](/images/2018-8-1/erlang_path.png)
环境变量配置完毕，可以打开cmd进行测试，输入命令：erl
若出现如下图的结果，则环境变量配置成功
![](/images/2018-8-1/cmd_success.png)
安装RabbitMQ
---
RabbitMQ下载地址：http://www.rabbitmq.com/download.html
![](/images/2018-8-1/rabbit_download.png)
下载完成后安装，可选择安装位置，如图所示
![](/images/2018-8-1/rabbitMQ_install.png)
![](/images/2018-8-1/rabbitmq_in.png)
安装RabbitMQ-Plugins
---
RabbitMQ-Plugins是一个管理界面，方便我们在浏览器界面查看RabbitMQ各个消息队列以及exchange的工作情况。
打开命令行cd进入rabbitmq的sbin目录(我的目录是：D:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.12\sbin)，输入：

	rabbitmq-plugins enable rabbitmq_management

稍等会会发现出现plugins安装成功的提示，默认是安装6个插件
在安装过程中也会出现错误，如下图所示：
![](/images/2018-8-1/rabbitmq_plugin.png)
解决方法是：
首先在命令行输入：rabbitmq-service stop
接着输入rabbitmq-service remove
再输入rabbitmq-service install
再输入rabbitmq-service start
重新输入rabbitmq-plugins enable rabbitmq_management
等待安装成功，当然，我在试验过程中发现不重新安装，直接rabbitmq-service start后也是可以的。
![](/images/2018-8-1/rabbitmq_pligins_error.png)
安装验证
---
在浏览器输入http://localhost:15672
进行验证,看到下面界面：
![](/images/2018-8-1/rabbitmq_test.png)
输入用户名：guest，密码：guest，进入管理界面
![](/images/2018-8-1/rabbitmq_pligins_success.png)
至此，安装完成！

本文另参考：
https://blog.csdn.net/hzw19920329/article/details/53156015