---
title: "Loadrunner12 使用基础教程"  # 文章标题
date: 2019-07-15T14:30:00+08:00  # 文章创建日期
draft: false  # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell"  # 文章作者
description: "Burpsuite中的信息外功能Burp Collaborator介绍"  # 文章描述
tags: ["性能测试","Loadrunner12"]  # 文章标签
categories: ["基础运维"]  # 文章分类
image: "/images/operate.png"
---



##  1. 安装（略）

- 程序安装包

HP_LoadRunner_12.02_Community_Edition_T7177-15059.exe

- 语言包

HP_LoadRunner_12.02_Community_Edition_Language_Packs_T7177-15062.exe

## 2. 使用简介

​	安装完成后，桌面上新增3个图标，分别是`Virtual User Generator`、 `Controller` 、`Analysis` 。

![](/images/loadrunner_img1.jpg)

### 2.1 录制脚本

​	打开`Virtual User Generator` ，使用组合键`Ctrl`+`N`打开如下对话框，选择`单协议`→`Web-HTTP/HTML`，填写脚本名称和保存位置后，点击创建。

![](/images/new_script.jpg)



> 创建后到达如下界面，可以发先操作分3类，分别为`vuser_init`、`Action`、`vuser_end`。他们的区别在于`vuser_init`只运行一次，在启动运行场景后最先运行；在`vuser_init`部分运行完后，`Action`根据运行场景的设置，多次并发运行；`Action`运行结束后，运行场景结束前运行`vuser_end`，也只运行一次。通常来说，`vuser_init`录制登录操作，`Action`部分录制需要测试并发的业务操作，`vuser_end`部分录制最后登出注销之类的释放资源的操作。

![](/images/operate.png)



​	录制vuser_init：选中vuser_init，按组合键`Ctrl`+`R`打开如下对话框，配置好url等相关参数后点击开始录制。Loadrunner会自动打开您选择的浏览器并访问您指定的URL。

![1551752491440](/images/vuser_init.png)



【注】如果出现下图错误，请确认网络是否正常，网络正常即可忽略该错误。

![1551686888809](/images/net_error.png)

​	输入用户名密码进行正常登录，

![1551752859139](/images/login.png)

​	登录完成后，将操作切换为Action，继续进行收件操作，如果还想测试其它事件场景，点击右侧加号，添加新的事件场景。需要测试的场景录制完成后，将操作切换至vuser_end，录制注销操作。录制完成后点击终止按钮结束录制。



### 2.2 运行脚本

​	录制完成后打开`Controller`，选中刚才录制的脚本，点击添加，然后点击`确定`按钮，进入场景配置界面。

![1551756887458](/images/run_script.png)

#### 2.2.1 场景配置

​	加载生成器，由于`Virtual User Generator`与`Controller`在同一计算机，所以添加一个localhost的生成器，确定即可。

![1551757110771](/images/load_generator.png)



​	配置全局计划，配置初始化方式、启动的vuser个数以及启动的时间、测试持续时间等。

![1551757309957](/images/global.png)



#### 2.2.2 监视服务器资源

​	点击底部的`运行`标签，切换至如下界面，根据服务器操作系统类型，选择`Windows资源`或`UNIX资源`，右击最右下角的统计图，在右击菜单中选择`添加度量`，到达服务器信息配置界面。

![1551757770639](/images/monitor.png)

#### 2.2.2.1 Linux

1. 在Linux服务器上安装并启动rstatd

```bash
apt-get install rstatd#安装rstatd
service rpcbind start #启动rpc服务
rpc.rstatd			  #启动rpc.rstatd服务
rpcinfo -p			  #查看服务状态和端口
```

![img](/images/rstatd_status.png)



2. 在Controller中添加资源视图，添加时，注意带上rstatd服务所监听的端口。

![1551773072125](/images/Linux.png)



#### 2.2.2.2 Windows

1. 确保机器B的Remote ProcedureCall(RPC)和Remote Registry Service服务开启。
2. 确保C盘默认共享开启。
3. 添加主机，输入用户名密码，用户必须为administrator或同权限用户。

![1551770475231](/images/windows.png)

4. 点击确定片刻后出现数据，表明资源监控配置成功。

![1551770718005](/images/windows_OK.png)



#### 2.2.3 开始场景

​	一切就绪，点击`开始场景`开始运行，等待测试结束。

![1551770816588](/images/start.png)

### 2.3 结果分析

​	场景测试完成后，点击如下图标，将打开`Analysis`，自动分析统计测试产生的数据。

![1551771453694](/images/Analysis.png)



​	`Analysis`打开后的界面如下，默认展示的统计图不包含系统资源的统计图。

![1551771964737](/images/analysis1.png)



​	按下组合键`Ctrl`+`A`来添加统计图

![1551772097547](/images/analysis2.png)



## 【参考链接】

[LoadRunner11实操压力测试](https://blog.csdn.net/Summer_Hanson/article/details/55504619)

[loadrunner监控linux服务器](https://www.cnblogs.com/seekwind/p/5817345.html)

[loadrunner监控Windows服务器](https://www.cnblogs.com/chengssblog/p/6646451.html)

