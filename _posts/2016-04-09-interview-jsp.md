---
layout: post
title:  "JSP常见面试问题"
categories: 学习笔记
tags: java jsp
excerpt: 面试者经常被问的一些jsp知识问题
---

#### 1、JSP四种范围是什么？区别是什么？

* **Page**: 指单单一页jsp page的范围；
* **Request**: 的范围只在一jsp页发出请求到另一页之间，随后这个属性失效；
* **Session**: 范围是用户和服务器连接的那段时间，用户与服务器断开属性就失效； 
* **Application**: 作用范围最大，在服务器一开始执行服务到服务器关闭为止。可能造成服务器负载过重。

#### 2、JSP有哪些内置对象?作用和分别是什么？

JSP共有以下9种基本内置组件（可与ASP的6种内部组件相对应）：

* **request** 用户端请求，此请求会包含来自GET/POST请求的参数 
* **response** 网页传回用户端的回应 
* **pageContext** 网页的属性是在这里管理 
* **session** 与请求有关的会话期 
* **application** servlet 正在执行的内容 
* **out** 用来传送回应的输出 
* **config** servlet的构架部件 
* **page** JSP网页本身 
* **exception** 针对错误网页，未捕捉的例外 

#### 3、JSP有哪些动作？作用分别是什么？

JSP共有以下6种基本动作 

* **jsp:include**：在页面被请求的时候引入一个文件。 
* **jsp:useBean**：寻找或者实例化一个JavaBean。 
* **jsp:setProperty**：设置JavaBean的属性。 
* **jsp:getProperty**：输出某个JavaBean的属性。 
* **jsp:forward**：把请求转到一个新的页面。 
* **jsp:plugin**：根据浏览器类型为Java插件生成OBJECT或EMBED标记 

#### 4、getAttribute()和setAttribute()的作用是什么？

#### 5、get和post的区别？

Form中的get和post方法，在数据传输过程中分别对应了HTTP协议中的GET和POST方法。二者主要区别如下：

* 1、Get是用来从服务器上获得数据，而Post是用来向服务器上传递数据。

* 2、Get将表单中数据的按照variable=value的形式，添加到action所指向的URL后面，并且两者使用“?”连接，而各个变量之间使用“&”连接；Post是将表单中的数据放在form的数据体中，按照变量和值相对应的方式，传递到action所指向URL。

* 3、Get是不安全的，因为在传输过程，数据被放在请求的URL中，而如今现有的很多服务器、代理服务器或者用户代理都会将请求URL记录到日志文件中，然后放在某个地方，这样就可能会有一些隐私的信息被第三方看到。另外，用户也可以在浏览器上直接看到提交的数据，一些系统内部消息将会一同显示在用户面前。Post的所有操作对用户来说都是不可见的。

* 4、Get传输的数据量小，这主要是因为受URL长度限制；而Post可以传输大量的数据，所以在上传文件只能使用Post（当然还有一个原因，将在后面的提到）。

* 5、Get限制Form表单的数据集的值必须为ASCII字符；而Post支持整个ISO10646字符集。

* 6、Get是Form的默认方法。

#### 6、写一个JSP页面，里面要包含一个表单、表单包含文本框、列表框、单选按扭、复选框。

#### 7、当前页面是a.jsp，用forward显示b.jsp的内容。

#### 8、什么是taglib?如何使用？有哪些方式？

1、问题：Tag究竟是什么？如何实现一个Tag？

一个tag就是一个普通的java类，它唯一特别之处是它必须继承TagSupport或者BodyTagSupport类。这两个类提供了一些方法，负责jsp页面和你编写的类之间的交互，例如输入，输出。而这两个类是由jsp容器提供的，无须开发人员自己实现。换句话说，你只需把实现了业务逻辑的类继承TagSupport或者BodyTagSupport，再做一些特别的工作，你的类就是一个Tag。并且它自己负责和jsp页面的交互，不用你多操心。

“特别的工作”通常有以下几个步骤： 　　1）提供属性的set方法，此后这个属性就可以在jsp页面设置。以jstl标签为例 ＜c:out value=""/＞，这个value就是jsp数据到tag之间的入口。所以tag里面必须有一个setValue方法，具体的属性可以不叫value。
　　例如：
　　
　　`setValue(String data){this.data = data;}`
　　
　　这个“value”的名称是在tld里定义的。取什么名字都可以，只需tag里提供相应的set方法即可。
　　
　　2）处理 doStartTag 或 doEndTag 。这两个方法是 TagSupport提供的。 还是以＜c:out value=""/＞为例，当jsp解析这个标签的时候，在“＜”处触发 doStartTag 事件，在“＞”时触发 doEndTag 事件。通常在 doStartTag 里进行逻辑操作，在 doEndTag 里控制输出。
　　
　　3）编写tld文件。
　　
　　4）在jsp页面导入tld
　　
　　这样，你的jsp页面就可以使用自己的tag了。
　　
　　通常你会发现自己绝大多数活动都集中在 doStartTag 或 doEndTag 方法里。确实如此，熟悉一些接口和类之后，写taglib很容易。正如《jsp设计》的作者所言：里面的逻辑稍微有点复杂，但毕竟没有火箭上天那么难。 
　　
　　2、一个简单的例子：OutputTag
　　

```
package com.eukire;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspWriter;
import javax.servlet.jsp.tagext.TagSupport;

/**
 * @author eukire
 */
public class OutputTag extends TagSupport{

    private String name=null; 
    
    public void setName(String name){
        this.name = name;
    }
    
    public int doStartTag() throws JspException{
        try{
            JspWriter out = pageContext.getOut();
            out.print("Hello! " + name);
        }catch (Exception e){ 
            throw new JspException(e);
        }
        return EVAL_PAGE;
    }
}
```

**简要说明：**

1、如何输出到jsp页面：
    调用JspWriter JspWriter out = pageContext.getOut();out.print......
    记住这个方法就可以了。
    
2、输出后如何作处理
    函数会返回几个值之一。EVAL_PAGE 表示tag已处理完毕，返回jsp页面。还有几个值，例如 EVAL_BODY_AGAIN 和EVAL_BODY_INCLUDE等，后面我们会作讨论编写tld 
```
＜?xml version="1.0" encoding="ISO-8859-1" ?＞
＜!DOCTYPE taglib PUBLIC "-//Sun Microsystems, Inc.//DTD JSP Tag Library 1.2//EN" "http://java.sun.com/dtd/web-jsptaglibrary_1_2.dtd"＞
＜taglib＞
    ＜tlib-version＞1.0＜/tlib-version＞
    ＜jsp-version＞1.2＜/jsp-version＞
    ＜short-name＞diego＜/short-name＞ 
    ＜!--OutputTag--＞
    ＜tag＞
        ＜name＞out＜/name＞
        ＜tag-class＞diegoyun.OutputTag＜/tag-class＞
        ＜body-content＞empty＜/body-content＞
        ＜attribute＞
            ＜name＞name＜/name＞
            ＜required＞false＜/required＞
            ＜rtexprvalue＞false＜/rtexprvalue＞
        ＜/attribute＞
    ＜/tag＞
＜/taglib＞
```

在WEB-INF下新建tlds文件夹，把这个文件取名为diego.tld，放到tlds文件夹下。路径应该这样：`WEB-INF\tlds\diego.tld`

关于tld的简单说明：

**short-name**：taglib的名称，也称为前缀。比如＜c:out value=""/＞ 里的“c”
**name**：tag的名字。例如＜c:out value=""/＞ 里的"out”，我们的类也取名为out，由于有前缀作区分，不会混淆
**tag-class**：具体的tag类。带包名
**body-content**：指tag之间的内容。例如＜c:out value=""＞ ...... ＜/c＞ 起始和关闭标签之间就是body-content。由于没有处理body-content，所以上面设为empty＜attribute＞里的name：属性名字。例如＜c:out value=""/＞里的value。名字可任意取，只要类里提供相应的set方法即可。
**required**：是否必填属性。
**rtexprvalue**：是否支持运行时表达式取值。这是tag的强大功能。以后我们会讨论。暂时设为false

编写jsp页面

```
＜%@ page language="java"%＞
＜%@ taglib uri="/WEB-INF/tlds/diego.tld" prefix="diego"%＞
＜html＞
＜body＞
#########Test Tag############
＜diego:out name="diegoyun"/＞
＜/body＞
＜/html＞
```

我的编程环境是eclipse+tomcat.启动服务器，如果一切按照上面步骤的话，就能看到 Test Tag: Hello! diegoyun 字样

最简单的tag就这么出来了。并不难,是不是?

#### 9、Jsp跳转有几种方式？分别是什么？

#### 10、JavaBean的范围？

`<jsp:useBean>`标签里有一属性scope，它用来设定JavaBean的范围，它的值只能为Page,request,session,application,不可为其它值。

使用不同的scope属性值，能在不用的范围共享JavaBean.

JSP中动态INCLUDE与静态INCLUDE的区别？ 

答：动态INCLUDE用jsp:include动作实现 

`<jsp:include page="included.jsp" flush="true" />`它总是会检查所含文件中的变化，适合用于包含动态页面，并且可以带参数 

静态INCLUDE用include伪码实现,定不会检查所含文件的变化，适用于包含静态页面 

`<%@ include file="included.htm" %> `

#### 12、两种跳转方式分别是什么?有什么区别? 

答：有两种，分别为： 

`<jsp:include page="included.jsp" flush="true"> `

`<jsp:forward page= "nextpage.jsp"/> `

前者页面不会转向include所指的页面，只是显示该页的结果，主页面还是原来的页面。执行完后还会回来，相当于函数调用。并且可以带参数.后者完全转向新页面，不会再回来。相当于go to 语句。
