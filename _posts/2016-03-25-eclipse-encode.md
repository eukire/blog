---
layout: post
title:  "Eclipse环境问题--编码"
categories: 学习笔记
tags:  Eclipse java tomcat
excerpt: 对于新手而言，没有哪个敢说开发过程中没有被IDE给坑过。这里就给大家讲述一些eclipse开发的常用设置。
---

* content
{:toc}

对于新手而言，没有哪个敢说开发过程中没有被IDE给坑过。这里就给大家讲述一些eclipse开发的常用设置。

# 一、全局编码设置

打开Window-->Preferences ，这里有当前eclipse的全局设置。

![2018/09/06/1.png](https://github.com/eukire/imgSrc/blob/master/2018/09/06/1.png?raw=true)

`General-->Workspace-->txt file ecoding` 设置自己需要的编码格式

虽然很简单，但是坑过了不少的人。比如项目是utf8，但是默认是gbk编码，这时候你新创建的文件默认编码就变成了gbk，在编译的时候往往就会出现因编码问题导致的失败构建

# 二、单文件设置

先看现象

![2018/09/06/2.png](https://github.com/eukire/imgSrc/blob/master/2018/09/06/2.png?raw=true)

有人会所是别人提交的代码编码有问题！首先这个观点不能说对错 ，只能说代码写的不完美。因为一般情况下jsp页面头部需要声明编码属性：

`<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>`

这样解决无疑是最佳的，但是有些文件是不需要编译，不能通过代码的方式改变编码的。这时候只需要两步就可以：1.右击该文件--->Properties---->编码改成UTF-8

![2018/09/06/3.png](https://github.com/eukire/imgSrc/blob/master/2018/09/06/3.png?raw=true)

# 三、Maven编译构建编码

关于Maven这里不多做讲解，如果想了解的话可以在Maven系列中详细查看。这里主要讲的是maven使用过程中编码的问题。以前遇到过一个问题，maven项目check出来以后，cmd命令构建打包的时候发现失败了。代码本身并没有问题，再看报错，是说某些类缺少符号。这是因为当项目没有指定编码的时候，cmd命令构建用的是机器本身的编码(GBK)，而我们大部分时候都是utf8的。所以这里我们来说说maven项目中需要指定编码的几个plugins。

**1.maven-compiler-plugin **

这个应该不用多说，只要是maven项目几乎都需要这个plugin ，这是完成build goal必须的步骤。很多童鞋一般也不会注意到这里的配置，所以我们先来看个例子：

``` xml
<plugin>
 <groupId>org.apache.maven.plugins</groupId>
 <artifactId>maven-compiler-plugin</artifactId>
 <version>3.2</version>
 <configuration>
     <source>1.8</source>
     <target>1.8</target>
     <encoding>UTF-8</encoding>
 </configuration>
 </plugin> 
```
这无疑是个常见的pom配置中的一段，指定编码其实非常简单，仅需要一行配置即可，顺便提下这个例子中还指定了jdk版本，开源项目里我们不建议这么做，因为这样会要求大家的版本跟你一样。但是在公司内部项目中却恰恰相反，指定jdk版本能做到内部统一规范，避免因为jdk版本引发的各种惨案。

**2.maven-surefire-plugin**

这个plugin应该关注的人就没有上面的那个多了，这是单元测试需要的plugin，也是经常出编码问题的一个地方。设置正确的编码如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.2</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
```

# 四、Tomcat编码设置

tomcat未设置编码，直接表现就是带中文的请求到后台以后无法正确解析。遇到这种情况先检查一下tomcat的配置。

在$catalina.home目录下conf文件夹，server.xml中有对tomcat全局编码的设置，找到并检查如下的配置：

`<Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/>`

# 写在最后：

这篇主要讲的是我们开发时经常遇到的编码问题，其实还有很多地方都有对编码的设置，比如前端meta、form，后台urlencoding、转码，数据库编码，部署发布时Apache、nginx等等。这里就不一一举例说明了。只要留个心，就能避免因为编码问题引发的各种悲剧。共勉!


