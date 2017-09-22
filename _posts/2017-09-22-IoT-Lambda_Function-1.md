---
layout: post
title:  "AWS Lambda(1)"
date:   2017-09-22 14:00:05
categories: IoT Cloud
excerpt: IoT Cloud-2-Cloud integration。
---

* content
{:toc}

# Create Lambda Function

---

## Prerequisite

### Setup AWS Env

####1.1	Install AWS CLI

- 安装Python3
- 安装awscli(pip3 install awscli)
- 添加AWS CLI到系统环境，编辑.bashrc, export PATH=~/.local/bin:$PATH	
	 
####1.2 Install Apex

- Download apex_linux_amd64
- cp apex_linux_amd64 /usr/local/bin/apex
- chmod +x /usr/local/bin/apex

####1.3 AWS credentials & Link Apex with AWS

- Create AWS IAM user and copy AWS access key & AWS secret access key
- In shell, run aws configure --profile <my-profile-name> 去添加IAM user到本机AWS configuration
- Export: export AWS_PROFILE="<my-profile-name>" 
- Export: export AWS_REGION="us-east-1"
- Run apex init, 提示输入project 信息
- Run apex deploy + functions目录下面的lambda app文件夹名完成Lambda部署


### Setup Node.js Env

####2.1	JavaScript基础

- 基本数据类型：number, boolean, string, null及undefined
- 复杂数据类型：array, function, 及object。访问复杂类型，访问的是对值的引用。
	- 例如： var a = [‘hello’, ‘world’]; b = a; b[0]=‘bye’; 那么 a[0] = ‘bye’。
- JavaScript同其他面向对象的语言一样有相应的构造器，很难判断数据的类型。
	- var a = ‘hello’; var b = new String(‘world’);  typeof a == ‘string’  typeof b == ‘object’; a instanceof String == false; b instanceof String == true;

####2.2	函数

- 函数可以作为引用存储在变量中，随后像其他对象一样进行传递：var a = function(a,b){}; console.log(a(1,2));
- s 