---
layout: post
title:  "Struts工作原理"
categories: 学习笔记
tags: java Struts
excerpt: Struts工作原理介绍
---

概述：

-------

Struts 使用 Model 2 架构。Struts 的ActionServlet 控制导航流。其他Struts 类，比如Action, 用来访问业务逻辑类。当 ActionServlet 从容器接收到一个请求，它使用URI (或者路径“path”) 来决定那个Action 将用来处理请求。一个 Action可以校验输入，并且访问业务层以从数据库或其他数据服务中检索信息。为校验输入或者使用输入来更新数据库，Action 需要知道什么指被提交上来。并不是强制每个Action 从请求中抓取这些值，而是由 ActionServlet 将输入绑定到JavaBean中。输入 bean是Struts ActionForm c类的类。ActionServlet 通过查找请求的路径可以决定使用哪个ActionForm，Action 也是通过同样的方法选取的。ActionForm 扩展org.apache.struts.action.ActionForm类。每个都必须以HTTP 响应进行应答。 通常, StrutsAction 并不自行加工响应信息，而是将请求转发到其他资源，比如JSP 页面。Struts 提供一个ActionForward 类，用来将一个页面的路径存储为逻辑名称。当完成业务逻辑后，Action 选择并向Servlet返回一个ActionForward。Servlet 然后使用存储在ActionForward 对象中的路径来调用页面完成响应。

Struts 将这些细节都绑定在一个ActionMapping 对象中。每个ActionMapping 相对于一
个特定的路径。当某个路径被请求时，Servlet 就查询ActionMapping 对象ActionMapping
对象告诉servlet，哪个Actions, ActionForms, 和 ActionForwards 将被使用。所有这些细节，关于Action， ActionForm， ActionForward， ActionMapping，以及其他一些东西，都在struts-config.xml 文件中定义。 ActionServlet 在启动时读取这个配置件，
并创建一个配置对象数据库。在运行时，Struts 应用根据文件创建的配置对象，而不是文件本身。

-------

在web应用启动时就会加载初始化ActionServlet,ActionServlet从struts-config.xml文件中读取配置信息,把它们存放到各种配置对象,例如:Action的映射信息存放在ActionMapping对象中.

当ActionServlet接收到一个客户请求时,将执行如下流程.

* (1)检索和用户请求匹配的ActionMapping实例,如果不存在,就返回请求路径无效信息;
     
* (2)如果ActionForm实例不存在,就创建一个ActionForm对象,把客户提交的表单数据保存到ActionForm对象中;
    
* (3)根据配置信息决定是否需要表单验证.如果需要验证,就调用ActionForm的validate()方法;
    
* (4)如果ActionForm的validate()方法返回null或返回一个不包含ActionMessage的ActuibErrors对象,就表示表单验证成功;
    
* (5)ActionServlet根据ActionMapping所包含的映射信息决定将请求转发给哪Action,如果相应的Action实例不存在,就先创建这个实例,然后调用Action的execute()方法;

* (6)Action的execute()方法返回一个ActionForward对象,ActionServlet再     把客户请求转发给ActionForward对象指向的JSP组件;
    
* (7)ActionForward对象指向JSP组件生成动态网页,返回给客户;