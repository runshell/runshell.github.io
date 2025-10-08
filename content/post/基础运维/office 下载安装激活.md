---
title: "office 下载安装激活" # 文章标题
date: 2025-05-05T14:30:00+08:00 # 文章创建日期
draft: false # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell" # 文章作者
description: "office下载、安装、激活全程安全无毒的方法介绍" # 文章描述
tags: ["office"] # 文章标签
categories: ["基础运维"] # 文章分类
---

## 下载

以下地址非官网地址，但下载地址为官网地址，可放心下载。选择适合的语言和版本。

```url
https://gravesoft.dev/office_c2r_links#chinese-simplified-zh-cn
```

## 安装

解压或挂载后，运行`setup.exe`或`Office\Setup64.exe`，等待安装完成。

## 激活

### 转换

将 office 转换成 vol 版本，才可使用 kms 激活。以管理员身份打开命令行(cmd 非 powershell)，执行：

```bat
@rem 进入office ospp.vbs文件所在目录
cd "C:\Program Files\Microsoft Office\Office16" || cd "C:\Program Files (x86)\Microsoft Office\Office16"
@rem ProPlus2024为office版本，修改为你需要激活的版本即可，有哪些版本可查看..\root\Licenses16 目录
for %x in (dir /b ..\root\Licenses16\ProPlus2024VL_*.xrm-ms) do cscript ospp.vbs /inslic:"%x"
```

### gvlk

从微软官方文档中获取对应版本的产品密钥，用于下一步激活

```url
https://learn.microsoft.com/zh-cn/office/volume-license-activation/gvlks
```

### kms 激活

```bat
@rem 注意跟换XJ2XN-FW8RK-P4HMP-DKDBV-GCVGB为上一步查询到的key
cscript ospp.vbs /inpkey:XJ2XN-FW8RK-P4HMP-DKDBV-GCVGB
@rem 设置kms服务器，此处以kms.03k.org为例
cscript ospp.vbs /sethst:kms.03k.org
@rem 激活
cscript ospp.vbs /act
```
