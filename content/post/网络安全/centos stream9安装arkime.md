---
title: "Centos Stream9安装Arkime" # 文章标题
date: 2024-07-15T14:30:00+08:00 # 文章创建日期
draft: false # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell" # 文章作者
description: "Burpsuite匹配中文" # 文章描述
tags: ["Arkime"] # 文章标签
categories: ["网络安全"] # 文章分类
image: "/images/Arkime_Logo.png"
---

## Arkime 简介

Arkime 是一个基于 Elasticsearch 的网络分析工具，用于捕获、存储和分析网络流量。它支持实时流量捕获和分析，以及历史流量的搜索和回放。Arkime 可以用于网络安全分析、流量监控和故障排除等场景。

## 下载包

```bash
wget https://mirror.ghproxy.com/https://github.com/arkime/arkime/releases/download/v4.3.0/arkime-4.3.0-1.el9.x86_64.rpm
```

## 安装依赖

```bash
yum install -y perl-libwww-perl perl-JSON perl-LWP-Protocol-https
```

## 安装 arkime

```bash
rpm -i arkime-4.3.0-1.el9.x86_64.rpm
```

## 阅读 readme

```bash
cat /opt/arkime/README.txt
```

## 查看网卡，清楚管理口网卡和用于接收镜像的网卡

```bash
ifconfig
```

## 执行配置脚本进行交互式配置

根据提示选择镜像网卡，输入密码等。配置过程会自动安装 elasticsearch，如果是内网机需手动安装，elasticsearch 可自行安装，本机部署建议监听 127.0.0.1

```bash
/opt/arkime/bin/Configure
```

## 启动服务

```bash
systemctl start elasticsearch.service
# 开机自启
systemctl enable elasticsearch.service
netstat -lnp | grep 9200
```

## 初始化 elasticsearch

```bash
/opt/arkime/db/db.pl http://127.0.0.1:9200 init
```

## 添加 web 管理员账号

```bash
/opt/arkime/bin/arkime_add_user.sh cbtdadmin "Admin User" fuzak0uling --admin
```

## 启动服务

```bash
systemctl start arkimecapture.service
systemctl start arkimeviewer.service
systemctl enable arkimecapture.service
systemctl enable arkimeviewer.service

netstat -lnp | grep 8005
```

## 出现 bug 查看日志

```bash
cat /opt/arkime/logs/viewer.log
cat /opt/arkime/logs/capture.log

# 出现 bug 查看 seLinux 开关

getenforce

# 主机防火墙配置

firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.x.x.x" port port=8005 protocol="tcp" accept'
firewall-cmd --runtime-to-permanent
```

## 可能缺失的文件

### 国内访问需要使用镜像站

```bash
wget " https://mirror.ghproxy.com/https://raw.githubusercontent.com/wireshark/wireshark/master/manuf"
mv manuf /opt/arkime/etc/oui.txt
```

### 通常无需访问镜像站

```bash
wget "https://www.iana.org/assignments/ipv4-address-space/ipv4-address-space.csv"
vi /opt/arkime/bin/arkime_update_geo.sh
mv ipv4-address-space.csv /opt/arkime/etc/

systemctl restart arkimecapture.service
```

## 优化配置

### 清理 60 天以前的流量日志

```bash
crontab -e
    0 0 * * * /opt/arkime/db/db.pl 127.0.0.1:9200 expire daily 60
```

### 配置 elasticsearch 水位线

```bash
curl -X PUT "http://127.0.0.1:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
"persistent": {
"cluster.routing.allocation.disk.watermark.low": "90gb",
"cluster.routing.allocation.disk.watermark.high": "50gb",
"cluster.routing.allocation.disk.watermark.flood_stage": "10gb",
"cluster.info.update.interval": "1m"
}
}'
```

### 配置删除 pcap 包保证空闲磁盘空间

```bash
vi /opt/arkime/etc/config.ini
freeSpaceG=200
```
