---
layout:     post
title:      "SpringBoot 整合webservice"
subtitle:   "哇咔咔."
date:       2019-07-17 23:06:59
author:     "Qing"
header-img: "img/post-bg-2019-07-17.jpg"
catalog: true
tags: 
    - Java
    - SpringBoot
    - WebService
---

> “Go on. ”

#### 初步
首先直接用idea构建springboot项目
在pom中添加以下依赖

```
<dependency>
         <groupId>org.apache.cxf</groupId>
         <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
         <version>3.2.4</version>
 </dependency>
```




#### 服务端代码
DemoService 


```
package com.zishu.service;

import com.zishu.constant.WebserviceConstant;

import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService(
        // 暴露服务名称
        name = "Demo_Service",
        //命名空间,一般是接口的包名倒序
        targetNamespace = "http://service.zishu.com"
)
public interface DemoService {

    /**
     * say hello
     * @param user
     * @return
     */
    @WebMethod
    public String sayHello(String user);
}
```


DemoServiceImpl


```
package com.zishu.service.impl;

import com.zishu.constant.WebserviceConstant;
import com.zishu.service.DemoService;
import org.springframework.stereotype.Service;

import javax.jws.WebService;
import java.util.Date;

/**
 * @author qun
 */
@WebService(
        //与接口中指定的name一致
        serviceName ="Demo_Service",
        // 与接口中的命名空间一致,一般是接口的包名倒
        targetNamespace ="http://service.zishu.com",
        // 接口地址
        endpointInterface = "com.zishu.service.DemoService"
)
@Service
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String user) {
        return user+"说：现在是北京时间"+new Date();
    }
}
```



CxfConfig  发布webservice配置


```
package com.zishu.config;

import com.zishu.service.DemoService;
import com.zishu.service.impl.DemoServiceImpl;
import org.apache.cxf.bus.spring.SpringBus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;

import javax.xml.ws.Endpoint;

/**
 * @author Qing
 */
@Configuration
@ImportResource({ "classpath:META-INF/cxf/cxf.xml" })
public class CxfConfig {


    @Bean
    public ServletRegistrationBean dispatcherServlet() {
        return new ServletRegistrationBean<>(new CXFServlet(), "/demo/*");
    }

    @Bean
    public SpringBus cxf() {
        return new SpringBus();
    }

    @Bean

    public DemoService demoService() {
        return new DemoServiceImpl();
    }

    @Bean
    public Endpoint endpoint() {
        EndpointImpl endpoint = new EndpointImpl(cxf(), demoService());
        endpoint.publish("/api");
        return endpoint;
    }
}
```



此时启动项目则可以访问到<http://localhost:8080/demo/api?wsdl> 并且看到以下wsdl


```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<wsdl:definitions xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" xmlns:tns="http://service.zishu.com" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:ns1="http://schemas.xmlsoap.org/soap/http" name="Demo_Service" targetNamespace="http://service.zishu.com">
<wsdl:types>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://service.zishu.com" elementFormDefault="unqualified" targetNamespace="http://service.zishu.com" version="1.0">
<xs:element name="sayHello" type="tns:sayHello"/>
<xs:element name="sayHelloResponse" type="tns:sayHelloResponse"/>
<xs:complexType name="sayHello">
<xs:sequence>
<xs:element minOccurs="0" name="arg0" type="xs:string"/>
</xs:sequence>
</xs:complexType>
<xs:complexType name="sayHelloResponse">
<xs:sequence>
<xs:element minOccurs="0" name="return" type="xs:string"/>
</xs:sequence>
</xs:complexType>
</xs:schema>
</wsdl:types>
<wsdl:message name="sayHelloResponse">
<wsdl:part element="tns:sayHelloResponse" name="parameters"> </wsdl:part>
</wsdl:message>
<wsdl:message name="sayHello">
<wsdl:part element="tns:sayHello" name="parameters"> </wsdl:part>
</wsdl:message>
<wsdl:portType name="Demo_Service">
<wsdl:operation name="sayHello">
<wsdl:input message="tns:sayHello" name="sayHello"> </wsdl:input>
<wsdl:output message="tns:sayHelloResponse" name="sayHelloResponse"> </wsdl:output>
</wsdl:operation>
</wsdl:portType>
<wsdl:binding name="Demo_ServiceSoapBinding" type="tns:Demo_Service">
<soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
<wsdl:operation name="sayHello">
<soap:operation soapAction="" style="document"/>
<wsdl:input name="sayHello">
<soap:body use="literal"/>
</wsdl:input>
<wsdl:output name="sayHelloResponse">
<soap:body use="literal"/>
</wsdl:output>
</wsdl:operation>
</wsdl:binding>
<wsdl:service name="Demo_Service">
<wsdl:port binding="tns:Demo_ServiceSoapBinding" name="DemoServiceImplPort">
<soap:address location="http://localhost:8080/demo/api"/>
</wsdl:port>
</wsdl:service>
</wsdl:definitions>
```



#### 客户端代码



```
package com.webservice.service;

import org.apache.cxf.endpoint.Client;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;

/**
 * @author Qing
 */
public class CfxClient {

    /**
     * 代理工厂的方式。需要拿到对方的接口地址
     */
    public static void client1() {
        //接口地址
        String address = "http://localhost:8080/demo/api?wsdl";
        //代理工厂
        JaxWsProxyFactoryBean factoryBean = new JaxWsProxyFactoryBean();
        //设置代理地址
        factoryBean.setAddress(address);
        //设置接口类型
        factoryBean.setServiceClass(DemoService.class);
        //创建一个代理接口实现
        DemoService service = (DemoService) factoryBean.create();
        //调用接口返回结果
        String result = service.sayHello("张天泽");
        System.err.println(result);
    }

    /**
     * 动态调用
     */
    public static void client2() {
        //创建动态客户端
        JaxWsDynamicClientFactory factory = JaxWsDynamicClientFactory.newInstance();
        Client client = factory.createClient("http://localhost:8080/demo/api?wsdl");
        //需要账户密码则加上账户密码
        try {
            Object[] invoke = client.invoke("sayHello", "江流儿");
            System.err.println(invoke[0]);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        client1();
        client2();
    }
}
```


注意点 |导致问题
---|---
EndpointImpl endpoint = new EndpointImpl(==springBus()==, demoService()); | row 当做形参传入时方法名必须为==cxf()== 否则：**No bean named 'cxf' available**
**targetNamespace** 和**endpointInterface** 必须填上，且service和serviceImpl 要对应 | 这两处不对应时在使用动态调用时，会**No operation was found with the name**





