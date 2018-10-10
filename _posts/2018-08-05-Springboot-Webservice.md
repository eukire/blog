---
layout: post
title:  "Springboot集成Webservice"
categories: 学习笔记
tags:  Springboot Webservice java
excerpt: Springboot集成Webservice的过程</br>记录了遇到的一些坑
---

* content
{:toc}

> 开源学习，自由转载，转载请注明出处！ 

本章节主要记录了Springboot集成Webservice的步骤以及遇到的一些问题

# 一、基本概念

Web Service平台需要一套协议来实现分布式应用程序的创建。任何平台都有它的数据表示方法和类型系统。要实现互操作性，WebService平台必须提供一套标准的类型系统，用于沟通不同平台、编程语言和组件模型中的不同类型系统。这些协议有：

**XML和XSD**
可扩展的标记语言（标准通用标记语言下的一个子集）是Web Service平台中表示数据的基本格式。除了易于建立和易于分析外，XML主要的优点在于它既与平台无关，又与厂商无关。XML是由万维网协会(W3C)创建，W3C制定的XML SchemaXSD　定义了一套标准的数据类型，并给出了一种语言来扩展这套数据类型。
Web Service平台是用XSD来作为数据类型系统的。当你用某种语言如VB. NET或C#　来构造一个Web Service时，为了符合Web Service标准，所有你使用的数据类型都必须被转换为XSD类型。如想让它使用在不同平台和不同软件的不同组织间传递，还需要用某种东西将它包装起来。这种东西就是一种协议，如 SOAP。

**SOAP**
SOAP即简单对象访问协议(Simple Object Access Protocol)，它是用于交换XML（标准通用标记语言下的一个子集）编码信息的轻量级协议。它有三个主要方面：XML-envelope为描述信息内容和如何处理内容定义了框架，将程序对象编码成为XML对象的规则，执行远程过程调用(RPC)的约定。SOAP可以运行在任何其他传输协议上。例如，你可以使用 SMTP，即因特网电子邮件协议来传递SOAP消息，这可是很有诱惑力的。在传输层之间的头是不同的，但XML有效负载保持相同。
Web Service 希望实现不同的系统之间能够用“软件-软件对话”的方式相互调用，打破了软件应用、网站和各种设备之间的格格不入的状态，实现“基于Web无缝集成”的目标。

**WSDL**
Web Service描述语言WSDL　就是用机器能阅读的方式提供的一个正式描述文档而基于XML（标准通用标记语言下的一个子集）的语言，用于描述Web Service及其函数、参数和返回值。因为是基于XML的，所以WSDL既是机器可阅读的，又是人可阅读的。

**UDDI**
UDDI 的目的是为电子商务建立标准；UDDI是一套基于Web的、分布式的、为Web Service提供的、信息注册中心的实现标准规范，同时也包含一组使企业能将自身提供的Web Service注册，以使别的企业能够发现的访问协议的实现标准。

## 调用RPC与消息传递

Web Service本身其实是在实现应用程序间的通信。我们有两种应用程序通信的方法：
RPC远程过程调用和消息传递。

使用RPC的时候，客户端的概念是调用服务器上的远程过程，通常方式为实例化一个远程对象并调用其方法和属性。RPC系统试图达到一种位置上的透明性：服务器暴露出远程对象的接口，而客户端就好像在本地使用的这些对象的接口一样，这样就隐藏了底层的信息，客户端也就根本不需要知道对象是在哪台机器上。

# 二、服务端集成步骤

## 1.加入ws相关依赖

在Springboot项目pom.xml中加入如下依赖：


```xml
<!-- webservice 集成 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
</dependency>
<dependency>
    <groupId>org.jdom</groupId>
    <artifactId>jdom2</artifactId>
</dependency>
<dependency>
    <groupId>wsdl4j</groupId>
    <artifactId>wsdl4j</artifactId>
</dependency>
```

备注：version管理都依赖spring-boot-starter-parent的version了

## 2.增加xsd文件描述
![5b5189655066b](https://user-images.githubusercontent.com/26571501/45066909-31921980-b0f3-11e8-8a44-c53d91965ae1.png)

在resource目录下增加文件夹ws用来放ws的所有xsd文件，同时增加agvJob.xsd，内容如下：


```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://www.panda-fa.com/wms/webservice"
           xmlns:wms="http://www.panda-fa.com/wms/webservice">
    <xs:element name="AgvJobRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="taskid" type="xs:int"/>
                <xs:element name="tasktype" type="xs:int"/>
                <xs:element name="startpostion" type="xs:string"/>
                <xs:element name="endpostion" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:element name="AgvJobResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="success" type="xs:boolean"/>
                <xs:element name="taskid" type="xs:int"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

定义了两个xml对象映射，分别是AgvJobRequest和AgvJobResponse

## 3.添加Springboot的启动支持

增加java类 WebServiceConfig.java


```java
package com.panda.wms.config;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.ws.config.annotation.EnableWs;
import org.springframework.ws.config.annotation.WsConfigurerAdapter;
import org.springframework.ws.transport.http.MessageDispatcherServlet;
import org.springframework.ws.wsdl.wsdl11.DefaultWsdl11Definition;
import org.springframework.xml.xsd.SimpleXsdSchema;
import org.springframework.xml.xsd.XsdSchema;
/**
 * 加入webservice支持
 *
 * @author J.y
 */
@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {
    /**
     * 配置webservice的根路径
     * <p>
     * /ws/*
     *
     * @param applicationContext
     *
     * @return
     */
    @Bean
    public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {
        MessageDispatcherServlet servlet = new MessageDispatcherServlet();
        servlet.setApplicationContext(applicationContext);
        servlet.setTransformWsdlLocations(true);
        return new ServletRegistrationBean(servlet, "/ws/*");
    }
    /**
     * agv的webservice接口
     * <p>
     * http://localhost:10030/wms/ws/agvJob.wsdl
     *
     * @param agvJobSchema
     *
     * @return
     */
    @Bean(name = "agvJob")
    public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema agvJobSchema) {
        DefaultWsdl11Definition wsdl11Definition = new DefaultWsdl11Definition();
        wsdl11Definition.setPortTypeName("agvJobPort");
        wsdl11Definition.setLocationUri("/ws");
        wsdl11Definition.setTargetNamespace(agvJobSchema().getTargetNamespace());
        wsdl11Definition.setSchema(agvJobSchema);
        return wsdl11Definition;
    }
    /**
     * xsd文件路径
     *
     * @return
     */
    @Bean
    public XsdSchema agvJobSchema() {
        return new SimpleXsdSchema(new ClassPathResource("ws/agvJob.xsd"));
    }
}
```

## 4.添加endpoint

添加ws接口实现类AgvJobEndpoint.java


```java
package com.panda.wms.webservice;
import com.alibaba.fastjson.JSON;
import com.panda.wms.vo.ws.AgvJobRequest;
import com.panda.wms.vo.ws.AgvJobResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ws.server.endpoint.PayloadEndpoint;
import org.springframework.ws.server.endpoint.annotation.*;
import javax.xml.transform.Source;
import java.util.UUID;
/**
 * agv任务上报
 *
 * @author J.y
 */
@Endpoint
public class AgvJobEndpoint {
    private final Logger logger = LoggerFactory.getLogger(AgvJobEndpoint.class);
    private static final String NAMESPACE_URI = "http://www.panda-fa.com/wms/webservice";
    /**
     * 任务上报
     *
     * @param request
     *
     * @return
     */
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "AgvJobRequest")
    @ResponsePayload
    public AgvJobResponse agvJob(@RequestPayload AgvJobRequest request) {
        String rId = UUID.randomUUID().toString().replaceAll("-", "");
        logger.info("receive agv job report beigin: rid=[{}],request=[{}]", rId, JSON.toJSON(request));
        AgvJobResponse response = new AgvJobResponse();
        response.setSuccess(true);
        response.setTaskid(request.getTaskId());
        logger.info("receive agv job report end : rid=[{}],response=[{}]", rId, JSON.toJSON(response));
        return response;
    }
}
```

其中：
1.namespace需要和xsd中的保持一致
2.AgvJobRequest.java和AgvJobResponse.java是接口出参和入参。需要手动加注解标示。效果如下：


```java
package com.panda.wms.vo.ws;
import javax.xml.bind.annotation.XmlRootElement;
import java.io.Serializable;
/**
 * Agv 调用入参 
 *
 * @author J.y
 */
@XmlRootElement(name = "AgvJobRequest", namespace = "http://www.panda-fa.com/wms/webservice")
public class AgvJobRequest implements Serializable {
    // 任务编号
    private Integer taskId;
    // 任务类型
    private int tasktype;
    // 起始位置
    private String startpostion;
    // 终点位置
    private String endpostion;
    // getter and setter
    ....
}
```

# 三、客户端调用过程

按照如上述步骤，启动springboot主程序即可。那么客户端如何调用呢？

## 1.访问wsdl

服务端部署成功后，可以访问wsdl，看到该接口的描述和规范：
eg:上述例子部署完成后，wsdl访问地址为：http://localhost:10030/wms/ws/agvJob.wsdl

结果如下：

![5b518d125cf73](https://user-images.githubusercontent.com/26571501/45067467-f5ac8380-b0f5-11e8-91bd-1f2982e90d83.png)

## 2.根据wsdl生成java调用代码

此处生成的代码的插件比较多，我这边选用的是Apache Axis2（关于Axis2，不清楚的最后会有说明）。
以IntelliJ为例

![5b518dab70251](https://user-images.githubusercontent.com/26571501/45067495-1d035080-b0f6-11e8-8865-3dfc08c7c162.png)

即可以生成client代码！

## 3.编写测试类

试着调用一下ws接口吧！


```java
public static void main(String[] args) throws RemoteException {
        com.panda_fa.wms.webservice.AgvJobRequest request = new com.panda_fa.wms.webservice.AgvJobRequest();
        request.setTaskid(1);
        request.setTasktype(0);
        request.setStartpostion("A0");
        request.setEndpostion("A1");
        AgvJobPortServiceStub stub = new AgvJobPortServiceStub();
        AgvJobResponse res = stub.agvJob(request);
        System.out.println(res.getTaskid());
    }
```

如此，客户端和服务端的测试工作都已完成

## 4.用http工具测试ws接口

ws接口本身是http调用，接口使用xml定义而已，也可以直接使用http工具调用服务端接口。
比如上面的测试main方法，可以写成：

![5b518e8c12501](https://user-images.githubusercontent.com/26571501/45067689-be8aa200-b0f6-11e8-8d90-a200a0c10ebb.png)

其中的body参数为


```xml
<?xml version='1.0' encoding='utf-8'?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:gs="http://www.panda-fa.com/wms/webservice">
<soapenv:Header/>
<soapenv:Body>
<gs:AgvJobRequest>
    <gs:taskid>1</gs:taskid>
    <gs:tasktype>0</gs:tasktype>
    <gs:startpostion>A0</gs:startpostion>
    <gs:endpostion>A1</gs:endpostion>
</gs:AgvJobRequest>
</soapenv:Body>
</soapenv:Envelope>
```

结果是一样的，只是没有解析过的xml而已。
ps，调用的时候需要设置httpHeader：**Content-Type:text/xml**

# 四、注意的点：

* 能访问到WSDL地址的时候，服务不一定可用，500错误的时候wsdl能正常显示出来
* ws的异常统一处理暂时没有看到，目前使用的也比较少，是直接在整个入口加了try/catch，否则异常抛出不可控
* namespace伴随xsd文件定义，wsdl定义、endpoint、入参和出参实体，如果调用结果如下错误，优先检查这个配置


```xml
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Header/>
  <SOAP-ENV:Body>
      <SOAP-ENV:Fault>
          <faultcode>SOAP-ENV:Server</faultcode>
          <faultstring xml:lang="en">No adapter for endpoint [public com.panda.wms.vo.ws.AgvJobResponse com.panda.wms.webservice.AgvJobEndpoint.agvJob(com.panda.wms.vo.ws.AgvJobRequest)]: Is your endpoint annotated with @Endpoint, or does it implement a supported interface like MessageHandler or PayloadEndpoint?</faultstring>
      </SOAP-ENV:Fault>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

* 关于更多xsd编写的语法问题，请参考 [XML语法规则](http://www.w3school.com.cn/schema/index.asp)
* Apache Axis2的使用：
* 1.下载最新版本的Axis2==>Apache [Axis2下载页]()
* 2.IntelliJ IDEA ——>Preferences——>Tools——>Webservice 设置：

![5b51934fa88cd](https://user-images.githubusercontent.com/26571501/45067799-3d7fda80-b0f7-11e8-8f3e-a0053c9ad599.png)


-------
> 自由转载，转载请注明出处！
