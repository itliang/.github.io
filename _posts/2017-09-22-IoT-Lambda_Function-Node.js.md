---
layout: post
title:  "AWS Lambda Function 开发"
date:   2017-10-12 14:00:05
categories: IoT
excerpt: 使用 Node.js 开发 AWS Lambda Function。
---

* content
{:toc}


## Prerequisite

### Setup AWS Env

#### 1.1	Install AWS CLI

- 安装Python3
- 安装awscli(pip3 install awscli)
- 添加AWS CLI到系统环境，编辑.bashrc, export PATH=~/.local/bin:$PATH	
	 
#### 1.2 Install Apex

- Download apex_linux_amd64
- cp apex_linux_amd64 /usr/local/bin/apex
- chmod +x /usr/local/bin/apex

#### 1.3 AWS credentials & Link Apex with AWS

- Create AWS IAM user and copy AWS access key & AWS secret access key
- In shell, run aws configure --profile <my-profile-name> 去添加IAM user到本机AWS configuration
- Export: export AWS_PROFILE="<my-profile-name>" 
- Export: export AWS_REGION="us-east-1"
- Run apex init, 提示输入project 信息
- Run apex deploy + functions目录下面的lambda app文件夹名完成Lambda部署

### Router
Router是一个孤立的中间件和路由的实例。Router可以被认为是一个”mini”的应用程序，仅能执行中间件和路由选择。
