---
title: "RHEL 8.10端口聚合" # 文章标题
date: 2025-05-06T10:58:00+08:00 # 文章创建日期
draft: false # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell" # 文章作者
description: "RHEL 8.10下端口聚合配置方法" # 文章描述
tags: ["RHEL"] # 文章标签
categories: ["基础运维"] # 文章分类
---

# 创建聚合组

```bash
# lacp 逐流负载
nmcli connection add type team con-name team0 ifname team0 config '{"runner": {"name": "lacp", "tx_hash": ["eth", "ipv4", "tcp", "udp"]}}'
```

# 配置聚合组

```bash
# 将接口eno2和eno3添加到聚合组team0中
nmcli connection add type team-slave con-name team0-port1 ifname eno2 master team0
nmcli connection add type team-slave con-name team0-port2 ifname eno3 master team0
```

# 配置 IP 地址

```bash
# 根据实际情况配置IP，配置静态IP
nmcli connection modify team0 ipv4.addresses 192.168.2.2/24
nmcli connection modify team0 ipv4.method manual
# 配置自动获取
# nmcli connection modify team0 ipv4.method auto
```

# 启动聚合组

```bash
nmcli connection up team0
nmcli connection up team0-port1
nmcli connection up team0-port2
```

# 设置 team0 连接为开机自动激活

```bash
nmcli connection modify team0 connection.autoconnect yes
nmcli connection modify team0-port1 connection.autoconnect yes
nmcli connection modify team0-port2 connection.autoconnect yes
```
