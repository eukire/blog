---
layout: post
title:  "Eclipse环境问题--Maven的配置"
categories: 学习笔记
tags:  Eclipse Maven java
excerpt: Eclipse配置Maven开发环境的一些步骤和注意事项
---

* content
{:toc}

# 一、设置Eclipse中maven环境变量

Eclipse中默认的conf配置是~/.m2/conf/setting.xml,一般我们都没有这个目录和文件，于是新建了一个默认的setting，在Eclipse中不设置的话就会引发一堆问题。此时我们需要手动指定settting位置。

![1459146855918_4](https://user-images.githubusercontent.com/26571501/45078967-dd535d80-b123-11e8-9695-432bf9990fc1.png)

设置完成保存以后 Update setting一下，配置就即时生效了。

同时注意下面的配置，推荐是勾上，如果每次打开Eclipse都去更新依赖，会导致每次启动环境非常慢。这在Eclipse最近的版本中已经改成了默认关闭，但是很早的版本中依然是默认更新依赖的，需要我们手动关闭掉选项。

![1459147116438_5](https://user-images.githubusercontent.com/26571501/45079579-864e8800-b125-11e8-8af5-08a3e05ed9e2.png)

# 二、新建maven项目

1.new --->other  选择maven project

![1459150707646_6](https://user-images.githubusercontent.com/26571501/45079743-de858a00-b125-11e8-8b79-c3414ce69036.png)

2.next -->勾选Create a simple project

![1459150765511_7](https://user-images.githubusercontent.com/26571501/45079888-30c6ab00-b126-11e8-9095-dd640fd831d5.png)

3.输入group id和 Artifact Id，以及最终打包的格式war或jar。（简单项目，具体maven坐标的含义就不深入解释了）

![1459150899150_8](https://user-images.githubusercontent.com/26571501/45080079-a763a880-b126-11e8-983c-3dc04207b52b.png)

# 三、导入maven项目

此处我们以导入git中的maven项目为例：
1.File-->Import-->Projects from Git
2.从本地git或远程获取一个项目导入到IDE

![1458694067080_2](https://user-images.githubusercontent.com/26571501/45080225-00cbd780-b127-11e8-82cc-709a8da5c9cd.png)