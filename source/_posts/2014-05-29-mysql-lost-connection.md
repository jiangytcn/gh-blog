---
layout: post
title: "MySQL Lost Connection"
description: ""
category: "Mysql"
tags: [mysql]
---
##Navicat连接mysql数据库连接错误Code 2013 Lost connection

###现象

2013 - Lost connection to Mysql Server at 'waiting for initial communication packet', system error:0

![Lost connection to Mysql Server at 'waiting for initial communication packet', system error:0][1]
###解决方法
在mysql的配置文件my.cnf 或my.ini 下添加
<pre><code>skip_name_resolve </code></pre>
使其在与mysql服务器建立连接时，跳过主机名的解析

[1]:http://jiangyt-github-io.qiniudn.com/Mysql%20%E8%BF%9E%E6%8E%A5%E5%A4%B1%E8%B4%A5.png
