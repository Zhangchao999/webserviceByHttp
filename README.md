---------
# TOC
[webservice是什么](#webservice是什么)<br>
[webservice运行原理](#webservice运行原理)<br>
[简单的示例](#简单的示例)<br>
[调用](#调用)<br>
[方法一](#方法一)<br>
[方法二](#方法二)<br>

----------



## webservice是什么
webservice的概念，这是一种技术，是一种跨越编程语言和操作系统平台的远程调用技术.<br>

* 跨越编程语言
> 就是服务器和客户端可以使用不同的语言(貌似现在的都是这样的)，例如：服务器使用Java语言，客户端可以使用其他的语言编写，反之亦然。<br>

* 跨越操作系统平台
> 就是指服务器端和客户端可以在不同的操纵系统上运行。<br>
* 远程调用
> 即一台计算机的应用可以调用其他计算机上的应用。<br>
简单的说，wenservice 可以让我们调用一些接口或者方法(已经有人写好的）例如关于天气，可以在 http://www.webxml.com.cn/WebServices/WeatherWebService.asmx?wsdl 中使用getWeatherbyCityName方法来获得城市的天气，这个方法就是我们在WSDL文件中找到的。

## webservice运行原理
webservice中，有三种技术：XML,SOAP，WSDL。<br>
* XML 
  > 简单就是我们所想的那个xml(常见于web.xml，spring-config.xml等等)。<br>
* SOAP 
  > 是一种协议（TCP/IP概念一样，就是协议，一种webservice中认可的规则），是基于XML的简易协议，可以使应用程序在HTTP之上进行信息交换，也就是说SOAP是用于访问网络服务的协议。<br>
* WSDL 
  > 相当于目录，可以同过这个文件查找到我们所用到的接口方法。<br>

## 简单的示例
所有的关于Java成语的示例一般都是 HelloWorld ， 或者 Hello... 的，我也不例外。<br>
在eclipse中新建Java项目，命名为HelloWeb。<br>
我的是这样的<br>
![HelloWeb项目](https://github.com/Zhangchao999/webserviceByHTTP/raw/master/pictures/1.png)
<br>
通过Endponint.publish(address,implement);发布服务，我发布的是 http://127.0.0.1:8888/HelloWeb/sayHello
可以在 地址后加?wsdl 来查看WSDL文件 例如：http://127.0.0.1:8888/HelloWeb/sayHello?wsdl 
这个服务是我现在发布的，只是用来测试的，如果你要访问肯定是不会访问到的。<br>
让我们来看看这个WSDL文件，顺便分析分析他的结构。
![WSDL文件](https://github.com/Zhangchao999/webserviceByHTTP/raw/master/pictures/2.png)
<br>
先来解释一下标签的意思：<br>
* service 
	> 相关端口的集合，包括其关联的接口、操作、消息等。
* Binding
	> 特定端口类型的具体协议和数据格式规范。
* portType
	> 服务端点，描述webservice可被执行的操作方法，以及相关的消息，通过Binding指向portType。
* message 
	> 定义一个操作方法的数据类型。
* type 
	> 定义webservice使用的全部数据类型。
<br>
如何查看：
WSDL文档应该从下往上看，以我们的为例：<br>
1、先看service标签，这个标签下的binding="tns:HelloServiceImplPortBinding" ,我们看HelloServiceImplPortBinding。<br>
2、往上找，找到binding标签，找到name 为 HelloServiceImplPortBinding 的，这里只有一个，但是第三方提供的或有很多个，
找到type="tns:HelloServiceImpl".<br>
3、继续往上找，找到portType 标签 ，找operation ,这就是我们可以用的方法，详细的数据可以在之上的message标签中找到详细的输入输出参数。<br>
也可以看图片，基本都标了出来。<br>

### 调用
重点来了，以上的部署及解释都是为了现在的调用。<br>
##### 方法一
使用JDK 自带的wsimport 生成webservice客户端。<br>
在eclipse中新建Java项目，项目名为getHelloWeb，使用cmd进入该项目的src文件下<br>
之后输入 wsimport -p com.zc.hello -s . http://127.0.0.1:8888/HelloWeb/sayHello?wsdl 命令。就会在src目录下生成可直接使用的客户端。<br>
cmd界面：<br>
![wsimport命令界面](https://github.com/Zhangchao999/webserviceByHTTP/raw/master/pictures/3.png)
形成的客户端：<br>
![客户端界面](https://github.com/Zhangchao999/webserviceByHTTP/raw/master/pictures/4.png)
<br>
这里，我们关注两个Java文件，一个是wsdl文档的portType元素的name 接口(HelloServiceImpl),一个是wsdl文档的service元素的name为名的接口(HelloServiceImplService)。
<br>
使用方式：<br>
新建一个Java文件(HelloClient.java)：<br>

```java 
package com.zc.hello;

public class HelloClient {
	public static void main(String[] args) {
		HelloServiceImpl helloServiceImpl = new HelloServiceImplService().getHelloServiceImplPort();
		String result = helloServiceImpl.sayHello(" ,哈哈哈我是输出");
		System.out.println("Result:"+result);
	}
}
```
```java
// 输出显示

Result:你好  ,哈哈哈我是输出
```
这样，我们就使用了自己发布的service，若是把该WSDL地址，告诉伙伴，他们也是可以访问到的(前提是你的先发布service),是不是很神奇。

#### 方法二
使用HttpURLConnection方式<br>
这个方法是自己编写客户端，所以会比较麻烦。<br>
使用HttpURLConnection 的方式可以完全访问HTTP资源，可以取代HttpGet和HttpPost类。可以分为如下六步：<br>
1、使用java.nat.URL 封装HTTP资源的URL，并使用openConnection方法获得HttpUrlConnection对象。<br>
2、设置请求方法，GET,POST等。<br>
3、设置输入流及其他权限。<br>
4、设置请求头。<br>
5、输入输出数据。<br>
6、关闭输入输出数据。<br>

查看SOAP的格式：<br>
使用eclipse自带的工具。
1、eclipse —— Window —— Show View —— TCP/IP Monitor<br>
点开 TCP/IP Monitor 的视图，倒三角，Properties 添加<br>
Local monitoring port: 这个是要监听的端口,改成一个没有使用过的(我这里是7777)。<br>
Host name : 127.0.0.1 <br>
Port : 原来的端口号,8888<br>
Type： 选择tcp/ip<br>
点击OK —— start   开始监听。<br>
把生成的客户端的 HelloServiceImplService.java 中的 url 全部由8888 改成 7777.<br>
之后，运行之前的HelloClient.java 文件，发起一次请求，并查看 request 与 response。目的是为了获取请求的SOAP Head。<br>
也可以使用 SOAPUI 这个软件来获得。<br>


```
// request
GET /HelloWeb/sayHello?wsdl HTTP/1.1
User-Agent: Java/1.8.0_162
Host: 127.0.0.1:7777
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
Connection: keep-alive

POST /HelloWeb/sayHello HTTP/1.1
Accept: text/xml, multipart/related
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://impl.service.zc.com/HelloServiceImpl/sayHelloRequest"
User-Agent: JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e
Host: 127.0.0.1:7777
Connection: keep-alive
Content-Length: 221


<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:sayHello xmlns:ns2="http://impl.service.zc.com/"><arg0> ,鍝堝搱鍝堟垜鏄緭鍑�</arg0></ns2:sayHello></S:Body></S:Envelope>
```

``` xml
// response
HTTP/1.1 200 OK
Date: Mon, 30 Jul 2018 04:52:47 GMT
Transfer-encoding: chunked
Content-type: text/xml;charset=utf-8

88c
<?xml version="1.0" encoding="UTF-8"?><!-- Published by JAX-WS RI (http://jax-ws.java.net). RI's version is JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e. --><!-- Generated by JAX-WS RI (http://jax-ws.java.net). RI's version is JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e. --><definitions xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" xmlns:wsp="http://www.w3.org/ns/ws-policy" xmlns:wsp1_2="http://schemas.xmlsoap.org/ws/2004/09/policy" xmlns:wsam="http://www.w3.org/2007/05/addressing/metadata" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:tns="http://impl.service.zc.com/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.xmlsoap.org/wsdl/" targetNamespace="http://impl.service.zc.com/" name="HelloServiceImplService">
<types>
<xsd:schema>
<xsd:import namespace="http://impl.service.zc.com/" schemaLocation="http://127.0.0.1:7777/HelloWeb/sayHello?xsd=1"></xsd:import>
</xsd:schema>
</types>
<message name="sayHello">
<part name="parameters" element="tns:sayHello"></part>
</message>
<message name="sayHelloResponse">
<part name="parameters" element="tns:sayHelloResponse"></part>
</message>
<portType name="HelloServiceImpl">
<operation name="sayHello">
<input wsam:Action="http://impl.service.zc.com/HelloServiceImpl/sayHelloRequest" message="tns:sayHello"></input>
<output wsam:Action="http://impl.service.zc.com/HelloServiceImpl/sayHelloResponse" message="tns:sayHelloResponse"></output>
</operation>
</portType>
<binding name="HelloServiceImplPortBinding" type="tns:HelloServiceImpl">
<soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"></soap:binding>
<operation name="sayHello">
<soap:operation soapAction=""></soap:operation>
<input>
<soap:body use="literal"></soap:body>
</input>
<output>
<soap:body use="literal"></soap:body>
</output>
</operation>
</binding>
<service name="HelloServiceImplService">
<port name="HelloServiceImplPort" binding="tns:HelloServiceImplPortBinding">
<soap:address location="http://127.0.0.1:7777/HelloWeb/sayHello"></soap:address>
</port>
</service>
</definitions>
0

HTTP/1.1 200 OK
Date: Mon, 30 Jul 2018 04:52:47 GMT
Transfer-encoding: chunked
Content-type: text/xml; charset=utf-8

f8
<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:sayHelloResponse xmlns:ns2="http://impl.service.zc.com/"><return>浣犲ソ  ,鍝堝搱鍝堟垜鏄緭鍑�</return></ns2:sayHelloResponse></S:Body></S:Envelope>
0


```

在这里我们可以获得SOAP的请求头为：
```xml
<?xml version="1.0" ?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
	<S:Body>
		<ns2:sayHello xmlns:ns2="http://impl.service.zc.com/">
			<arg0> ,鍝堝搱鍝堟垜鏄緭鍑�</arg0>
		</ns2:sayHello>
	</S:Body>
</S:Envelope>
```
乱码的是我们输入的中文。<br>

HelloClient2.java 代码如下：<br>
```java
package com.zc.hello;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class HelloClient2 {
	/**
	 * 发送SOAP格式
	 * @param name 要显示的信息
	 * @return
	 */
	public static String getXML(String name) {
		String soapXML = "<?xml version=\"1.0\" ?>\r\n" + 
				"<S:Envelope xmlns:S=\"http://schemas.xmlsoap.org/soap/envelope/\">" + 
				"<S:Body>" + 
				"<ns2:sayHello xmlns:ns2=\"http://impl.service.zc.com/\">" + 
				"<arg0> "+name+"</arg0>" + 
				"</ns2:sayHello>" + 
				"</S:Body>" + 
				"</S:Envelope>";
		return soapXML;
	}
	public static void main(String[] args) throws IOException {
		// 1、创建服务地址，并获得HttpURLConnection对象
		URL url = new URL("http://127.0.0.1:8888/HelloWeb/sayHello?wsdl");
		HttpURLConnection connection = (HttpURLConnection)url.openConnection();
		// 2、设置请求方法，数据格式
		connection.setRequestMethod("POST");
		connection.setRequestProperty("content-type", "text/xml;charset=UTF-8");
		// 3、设置输入输出权限
		connection.setDoInput(true);
		connection.setDoOutput(true);
		// 4、 设置请求头
		String soapXML = getXML("哈哈哈，你也好！");
		// 5、设置请求头
		OutputStream os = connection.getOutputStream();
		os.write(soapXML.getBytes());
		int responseCode = connection.getResponseCode();
		if(200 == responseCode) {
			InputStream is = connection.getInputStream();
			InputStreamReader isr = new InputStreamReader(is, "utf-8");
			BufferedReader br = new BufferedReader(isr);
			StringBuffer sb = new StringBuffer();
			String temp = null;
			while(null != (temp = br.readLine())) {
				sb.append(temp);
			}
			System.out.println(sb.toString());
			// 6、关闭流
			br.close();
			isr.close();
			is.close();
		}
	}
}
```
```
// 输出为：
<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:sayHelloResponse xmlns:ns2="http://impl.service.zc.com/"><return>你好  哈哈哈，你也好！</return></ns2:sayHelloResponse></S:Body></S:Envelope>
```
这样，我们就获取到webservice 返回的文件了。
