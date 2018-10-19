---
layout: post
title:  "Mysql调优--connections设置"
categories: 学习笔记
tags:  Mysql sql
excerpt: Mysql配置方面的优化，连接数方面的设置
---

* content
{:toc}


> 开源学习，自由转载，转载请注明出处！ 

# 1.问题描述

先来看个常见的报错信息：

`ERROR 1040 (08004): Too many connections`

很明显的数据库连接数超了。有些童鞋可能至今没有遇到过，原因我后面会说。

网上给的解决方案很多也很典型：

## 临时解决：


```
set GLOBAL max_connections=1000; 
```

设置一下可以临时解决该问题。但是重启后又会变成原来的值。

## 永久解决：

配置数据库配置文件：`my.cnf`

```
max_connections=1000
```

如果是mariadb的话，有默认打开文件数的限制。需要配置`/usr/lib/systemd/system/mariadb.service`，添加如下两行：

```
LimitNOFILE=10000
LimitNPROC=10000
```

重新加载系统服务，并重启mariadb服务

```
systemctl --system daemon-reload 
systemctl restart mariadb.service
```

# 2.引申学习

先安利大家一个网站，计算mysql内存的：[mysqlcalculator](http://www.mysqlcalculator.com/)

![2018/09/12/1.png](https://github.com/eukire/imgSrc/blob/master/2018/09/12/1.png?raw=true)

如图，我们可以看到Mysql默认安装的运行内存需求是**576.2 MB**。其中，**max_connections**默认设置是150.

可能有人对150没有具体的认识，我们可以这样来看：

```
......
initial-size: 1
min-idle: 3
max-active: 20
......
```

这是代码中的jdbc配置的一部分，其中：

* initial-size 初始化连接池的数量
* min-idle 最小链接数量
* max-active 最大链接数量

这三个配置算是中低等了，保守估计双机大约最多可以支持10万PV吧。也就是说满载的状态下，10万PV连接数在40，其实这对mysql默认值来说并不算太大。
但是现在分布式系统盛行，一个应用可能被拆作4-5个微服务单独部署，这样的话，全部双机部署最大就是**20x5x2=200**，已经超出了默认上限。
这时候拓展最大连接数就有意义了。

再有我们可以看到 一个**max_connections**内存的构成：

```
sort_buffer_size 2M
read_buffer_size 0.128M
read_rnd_buffer_size 0.256M
join_buffer_size 0.128M
thread_stack 0.196M
binlog_cache_size 0M
```

也就是说，默认一个数据库连接需要使用2.7Mb的内存资源，而实际生产中DBA对此配置还会适当微调，一般也是只大不小。所以一台8G内存的数据库服务器，不考虑交换内存，最大也就支持3000左右的最大连接数。这也是比较常见的一种数据库瓶颈，虽然根源一般是应用使用数据库连接不当引起的，所以解决方案一般也是由应用去保证释放一些不必要的数据库连接。

> 开源学习，自由转载，转载请注明出处！ 



 


