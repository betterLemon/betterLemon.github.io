---
title: cmd python乱码问题
date: 2023-03-08 09:24:00
tags:
---

### 将CMD终端改为UTF-8格式

#### 命令介绍：

- ​	chcp 65001  #换成utf-8代码页 

- ​	chcp 936    #换成默认的gbk 

- ​	chcp 437    #美国英语 


​    一般默认为gbk，若要修改成 utf-8，则需要： 

#### 修改办法

1. cmd窗口输入： 

   ```bash
   chcp 65001 
   ```

2. 修改cmd属性： 

   选择字体为“Lucida Console”
