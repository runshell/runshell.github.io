---
title: "Burp中匹配中文"  # 文章标题
date: 2018-07-15T14:30:00+08:00  # 文章创建日期
draft: false  # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell"  # 文章作者
description: "Burpsuite匹配中文"  # 文章描述
tags: ["网络安全","Burpsuite"]  # 文章标签
categories: ["网络安全"]  # 文章分类
image: "/images/image1.png"
---



**问题：**Burp中有很多地方可以进行正则匹配，比如Instruder模块中筛选响应包，proxy模块中匹并配替换字符串。中文在匹配的时候，添加进匹配列表就变身了，关键是与数据包内的相应字符不能匹配。

**解决办法：**

1. 在user option中设置字符集(character sets)为显示原始字节流(Display as raw bytes);

![image1](..//images/image1.png)

2. 在响应包中复制要匹配的中文，显示的是乱码；

![image2](..//images/image2.png)



3. 将复制的乱码粘贴到添加匹配字符串的地方。

![image3](..//images/image3.png)