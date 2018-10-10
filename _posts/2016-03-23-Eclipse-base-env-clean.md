---
layout: post
title:  "Eclipse环境问题基础篇--clean&update"
categories: 学习笔记
tags:  Eclipse Maven
excerpt: Eclipse开发环境的三板斧
---

* content
{:toc}

首先，耐心看完这三步，能解决我们开发过程中遇到的90%的环境问题

1. 右击项目工程--->refresh
2. project--->clean
3. server右击 --->clean


正如网管的重启大法一样，Eclipse大部分环境问题都可以通过这三步能解决！
 
 如果上面的三步没有解决问题，我们再继续往下看。
 
**maven update**

 maven项目中经常遇到这样逼死强迫症的情况，项目飘红，但是具体文件却不报错：
 
![1459144592690_2](https://user-images.githubusercontent.com/26571501/45081559-0a0a7380-b12a-11e8-8fc6-ff64becff5c4.png)
 
解决方法很简单：**ALT+F5！**

至于原因 ，就是pom文件有更新项目依赖没有及时刷新，或者本地Eclipse关闭了auto build.

![1459144743683_3](https://user-images.githubusercontent.com/26571501/45081590-1c84ad00-b12a-11e8-83e8-caee848a0639.png)


