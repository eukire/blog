---
layout: post
title:  "postman进阶--将返回结果当做其他请求的变量"
categories: 学习笔记
tags: java postman
excerpt: 对于有权限限制的系统而言，很多接口需要鉴权，比如用登陆的token放到header里面，下一个接口才能正常访问。那么Postman如何将第一个结果的返回值应用到第二个接口中去？
---

> Postman Makes API Development Simple.

![2018/10/12/1.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/1.png?raw=true)

[Postman](https://www.getpostman.com/) 一般是开发自测接口用的，因为能团队协作，有的甚至已经当做接口文档标准来使用了。

她可是允许用户发送任何类型的 HTTP 请求，例如 GET，POST，HEAD，PUT、DELETE等，并且可以允许任意的参数和 Headers。
她支持不同的认证机制，包括 Basic Auth，Digest Auth，OAuth 1.0，OAuth 2.0等。
她还可以响应数据是自动按照语法格式高亮的，包括 HTML，JSON和XML。
她还...

本文对于post的各种好处不再一一赘述，主要想说的是如何在一个请求入参里面使用另外一个请求的结果。

### 使用场景

一个常用的场景是这样的：对于有权限限制的系统而言，很多接口需要鉴权，比如用登陆的token放到header里面，下一个接口才能正常访问。或者连续测试一堆接口时，插入接口测完了返回的id，会在下一个修改或者删除接口中使用等等。

### 解决方案

针对上述场景，可以用下面的方法来配置解决。

#### 环境变量、变量设置

**manage environment** 里面添加env

![2018/10/12/2.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/2.png?raw=true)

例如可以添加localhost、dev、test、prd等不同的环境变量，当然如果有需要也可以添加global。

在**localhost**里面设置变量 `token` ，注意这个token值是空的 后面通过脚本写入

![2018/10/12/3.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/3.png?raw=true)

#### 前置接口脚本设置

准备前置接口：

![2018/10/12/4.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/4.png?raw=true)

我这边用的是一个简单的登陆接口，登陆成功后返回登陆token，其他接口调用时，需要用这个参数去校验用户身份。

在 **Tests** 标签内写脚本：

```js
if(pm.response.to.have.status(200) && pm.expect(pm.response.text()).to.include("token")){
    var jsonData = pm.response.json();
    pm.environment.set("token", jsonData.data.token);
}
```

如图

![2018/10/12/5.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/5.png?raw=true)

#### 测试调用结果

调用前置接口后，我们查看变量看看：

![2018/10/12/6.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/6.png?raw=true)

变量已经成功设置，接下来就可以调用普通的需要登陆的接口了。

![2018/10/12/7.png](https://github.com/eukire/imgSrc/blob/master/2018/10/12/7.png?raw=true)

以上！




