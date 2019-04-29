---
title:       "webservice-wsdl理解"
subtitle:    ""
description: ""
date:        2019-04-02
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-01-may-day-jiulonghu/snowmountain.jpg"
tags:        ["RPC", "wsdl"]
categories:  ["Tech" ]
---

#### wsdl是什么

webservice定义语言， 对应.wsdl文档，一个webService会对应一个唯一的wsdl文档，定义了客户端与服务端发送请求和响应的数据格式和过程。

#### wsdl的结构图文档理解 

转载地址：  https://lixh1986.iteye.com/blog/2426291 

元素binding显示出，暴露的访问接口方法。 

```xml
<?xml version="1.0" encoding="UTF8" ?>  
  
<wsdl:definitions targetNamespace="http://www.57market.com.cn/HelloService"   
    xmlns:soapenc12="http://www.w3.org/2003/05/soapencoding"   
    xmlns:tns="http://www.57market.com.cn/HelloService"   
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"   
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"   
    xmlns:soap11="http://schemas.xmlsoap.org/soap/envelope/"   
    xmlns:wsdlsoap="http://schemas.xmlsoap.org/wsdl/soap/"   
    xmlns:soapenc11="http://schemas.xmlsoap.org/soap/encoding/"   
    xmlns:soap12="http://www.w3.org/2003/05/soapenvelope">  
  
  
/**  
 * types元素,定义了交换信息的数据格式。  
 * 为了实现最大的互操作性（interoperability）和平台中立性（neutrality），  
 * WSDL选用XML Schema DataTypes简称XSD作为标准类型系统，并将它作为固有类型系统。  
 *  
 * 下面是数据定义部分，该部分定义了两个元素，一个是sayHello，一个是sayHelloResponse：  
 * sayHello：定义了一个复杂类型，仅仅包含一个简单的字符串，将来用来描述操作的参入传入部分；  
 * sayHelloResponse：定义了一个复杂类型，仅仅包含一个简单的字符串，将来用来描述操作的返回值；  
 */  
  
  
<wsdl:types>  
  
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"   
                attributeFormDefault="qualified"   
                elementFormDefault="qualified"   
                targetNamespace="http://www.57market.com.cn/HelloService">  
                  
  
        <xsd:element name="sayHello">  
            <xsd:complexType>  
                <xsd:sequence>  
                    <xsd:element maxOccurs="1" minOccurs="1" name="in0" nillable="true" type="xsd:string" />  
                </xsd:sequence>  
            </xsd:complexType>  
        </xsd:element>  
  
  
        <xsd:element name="sayHelloResponse">  
            <xsd:complexType>  
                <xsd:sequence>  
                    <xsd:element maxOccurs="1" minOccurs="1" name="out" nillable="true" type="xsd:string" />  
                </xsd:sequence>  
            </xsd:complexType>  
        </xsd:element>  
  
   
    </xsd:schema>  
  </wsdl:types>  
  
   
  
/**  
 * message元素是交换信息的抽象定义。  
 * 需要分别定义用于作为输入参数的消息和输出参数的消息。  
 *  
 * 定义了两个消息 sayHelloRequest 和 sayHelloResponse：  
 * sayHelloRequest：sayHello操作的请求消息格式。由一个消息片断（part）组成。  
 * 片段的名字为 parameters, 片段的元素是我们前面定义的types中的元素；  
 * sayHelloResponse：sayHello操作的响应消息格式。由一个消息片断（part）组成。  
 * 片段的名字为 parameters, 片段的元素是我们前面定义的types中的元素；  
 *  
 * 如果采用RPC样式的消息传递，只需要将文档中的element元素应以修改为type即可。  
 *  
 * - message：用來定義消息的結構  
 * - part：指定引用types中定義的標籤片斷  
 *  
 */  
  
<wsdl:message name="sayHelloRequest">  
     <wsdl:part name="parameters" element="tns:sayHello" />  
</wsdl:message>  
  
<wsdl:message name="sayHelloResponse">  
     <wsdl:part name="parameters" element="tns:sayHelloResponse" />  
</wsdl:message>  
  
   
  
/**  
  
 * portType元素中定义了Web服务的操作。一些抽象操作的集合。  
 * 操作定义了输入和输出数据流中可以出现的XML消息。  
 * 每个操作关联一个输入消息和一个输出消息。  
  
 * 这里包含一个操作（operation）：sayHello方法，  
 * 同时包含input和output表明该操作是一个 请求／响应模式，  
 *   
 * input请求的消息体是前面定义的 sayHelloRequest。  
 * output响应的消息体是前面定义的 sayHelloResponse。  
  
 *  - portType：用來定義服務端的SEI  
 *  - operation：用來指定SEI中的處理請求的方法  
 *  - input：指定客戶端應用傳過來的數據，會引用上面的而定義的message  
 *  - output:指定服務端返回給客戶端的數據，會引用上面的而定義的message  
 */  
  
<wsdl:portType name="HelloServicePortType">  
  
    <wsdl:operation name="sayHello">  
        <wsdl:input name="sayHelloRequest" message="tns:sayHelloRequest" />  
        <wsdl:output name="sayHelloResponse" message="tns:sayHelloResponse" />  
    </wsdl:operation>  
  
</wsdl:portType>  
  
   
  
/**  
 * binding 元素描述特定服务接口的协议、数据格式、安全性和其它属性。  
 * 针对operation和portType中使用的message指定实际的协议和数据格式规范。  
 *  
 * binding:用于定义SEI的实现类  
 *  - type：引用上面的 portType  
 *  - style：document 绑定的数据是一个document（XML）  
 *  - operation：用来定义实现的方法  
 *  - input: 指定客户端应传过来的数据  
 *  - output：指定服务器端返回客户端的数据  
 *  - literal：文本数据  
 */  
  
<wsdl:binding name="HelloServiceHttpBinding" type="tns:HelloServicePortType">  
  
        <wsdlsoap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>  
  
        <wsdl:operation name="sayHello">  
            <wsdlsoap:operation soapAction="" />  
            <wsdl:input name="sayHelloRequest">  
                <wsdlsoap:body use="literal" />  
            </wsdl:input>  
            <wsdl:output name="sayHelloResponse">  
                <wsdlsoap:body use="literal" />  
            </wsdl:output>  
        </wsdl:operation>  
  
</wsdl:binding>  
  
   
 
/**  
 * service元素包含一组port元素。  
 *  
 * port将端点与来自服务接口定义的 binding 元素关联起来。  
 * port指定一个绑定的地址，这样定义一个通信的终端。  
 *  
 * - service：一個webservice的容器  
 * - name：它用以指定一個服務器端處理請求的入口（就是SEI的實現）  
 * - binding：引用上面定义的 binding  
 * - address:当前 webservice 的请求地址  
 *  
 */  
  
<wsdl:service name="HelloService">  
  
    <wsdl:port name="HelloServiceHttpPort" binding="tns:HelloServiceHttpBinding">  
        <soap:address location="http://localhost:8080/xfire/services/HelloService" />  
    </wsdl:port>  
  
</wsdl:service>  
  
   
  
  
</wsdl:definitions>  

```

###   

#### 一个webService的请求流程

如何发布一个webService:   

1. 定义SEI    @webservice  @webMethod
2. 定义SEI实现
3. 发布： Endpoint.publish(url,SELimplObject)

如何请求一个webService: 

1. 根据wsdl文档生成客户端代码.  jdk/cxf
2. 根据生成的代码调用webService







