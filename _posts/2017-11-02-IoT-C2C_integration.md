---
layout: post
title:  "Cloud to Cloud integration"
date:   2017-11-02 13:40:05
categories: IoT
excerpt: Cloud to Cloud integration solution。
---

* content
{:toc}


# Introduction
Cloud integration is the process of configuring multiple application programs to share data in the cloud. In a network that incorporates cloud integration, diverse applications communicate either directly or through third-party software.

云集成是配置多个应用程序在云之间共享数据的过程。在整合云集成的网络中，不同的应用可以直接或者通过第三方软件相互沟通。

# Mission & Goal

通过一个客户端无缝地控制自己云和第三方平台上的设备。整合一个第三方的设备到我们的平台中，对最终用户来说，当这个设备出现在第三方平台的客户端以后，在我们的客户端里面经过成功的OAuth流程后，这个设备也应该出现在我们的客户端中。

# 如何实现

为了实现我们的目标，我们需要对目前的控制端APP, Cloud端的Service做些功能的增强，并且通过引入Webhook机制和AWS Lambda Function来集成第三方的云。下面我会做详细的介绍。

## 增强现有的云服务抽象
我们设计了两种第三方的云集成方案，通过创建AWS Lambda function 或者一个带有RESTful API接口的Webhook。两种方案各有利弊，具体的区别在于Serverless & Webserver 这两种体系架构之间的差异。在自身的云服务上通过抽象出Connector的概念来配置Lambda function的arn(rn:aws:lambda:****)和Webhook的监听URL(Https://iotivity.top)。

对于ConnectorInfo，主要的抽象实现如下：

	class ConnectorInfo{
		String ConnectorName; 		//名称
		String ConnectorID;		  	//唯一ID
		String ManufacturerName; 	//生成者名称，个人或者是公司名称
		String Description;      	//对Connector的描述
		Boolean MultiInstance;   	//是否允许你的Connector被客户端安装到不同的Location下面
		String ConnectorType;			//AWS Lambda 或者 Webhook
		String TartgetARN;				//Lambda arn 或者 Webhook URL
		String OwnerID;						//创建者的ID
		String UpdateDate;				//最后更新或者创建的日期
	}

云对云的集成，其中数据的抽象是最耗时和难度最大的工作，通过DeviceProfile来定义平台中的设备信息。对于一个设备,又可以抽象出几种不同的能力Capability,每种Capability都会包含对它的CommandList及状态的信息列表AttributeList。其主要的抽象定义如下：

	class DeviceProfile{
		String name;
		ArrayList<String> CapabilityList;
	}
	class Capability{
		String name;				//名称
		String status;				//可用或者测试阶段
		ArrayList<String> commands; //Command列表和Key-value对
		ArrayList<String> attributes; //Attribute列表和Key-Value对
		boolean status;				//Capability是否被外部开发者使用
		String id;					//唯一标识Id
		String ocfResourceType; 	//对应的OCF Resource Type
	}
Cloud端使用OCF的规范是比较好的选择，目前已经采用RAML和JSON文件格式定义了将近300种的ResourceType。详细内容可以参考[OCF IoT DataModel](https://oneiota.org/documents?filter%5Bmedia_type%5D=application%2Framl%2Byaml).

### Webhook

Webhook,就是人们常说的钩子，通过定制Webhook来检测Cloud上的各种事件，比如设置了一个监测 push 事件的 Webhook，那么每当有了任何push操作，这个Webhook都会被触发，这时Cloud就会发送一个 HTTP POST请求到配置好的地址。

#### 主机要求

使用Node.js创建HTTPS服务器。首先使用openssl生成证书文件：

1： 数字证书是一个经证书授权中心数字签名的包含公开密钥拥有者信息及公开秘钥的文件。实际应用中，一般都不会找CA去签名(因为收费)，可以做一个自签名的证书文件，即自己生成一对密钥，然后再用自己生成的另外一对密钥对密钥进行签名。

2： 密钥文件的格式用OpenSSL生成的有PEM和DER两种格式，PEM是将密钥用base64编码表示出来，是一串英文字母。DER是二进制的密钥文件。X509是通用的证书文件格式。

	openssl genrsa -out ca.key.pem 2048 //构建根证书私钥
	req -new -key ca.key.pem -out certrequest.csr  //生成根证书签发申请文件
	openssl x509 -req -days 10000 -sha1 -signkey ca.key.pem -in certrequest.csr -out ca.key.cer //签发根证书/自行签发根证书
		x509        签发X.509格式证书命令
 		-req        表示证书输入请求
	 	-days       表示有效天数,这里为10000天
	 	-shal       表示证书摘要算法,这里为SHA1算法
	 	-signkey    表示自签名密钥,这里为ca.key.pem
	 	-in         表示输入文件,这里为certrequest
	 	-out        表示输出文件,这里为ca.key.cer



#### 响应Webhook的服务器

为了响应 Webhook 发出的请求，需要先实现一个响应服务器。



### Lambda
