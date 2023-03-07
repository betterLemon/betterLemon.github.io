---
title: centos服务器防火墙端口操作
date: 2023-03-07 10:09:41
tags:
---

```bash
# 增加端口开放
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --zone=public --add-port=3399/tcp --permanent
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --zone=public --add-port=9092/tcp --permanent
firewall-cmd --zone=public --add-port=2900/tcp --permanent
firewall-cmd --zone=public --add-port=7001/tcp --permanent
firewall-cmd --reload 
firewall-cmd --zone=public --list-ports


```

