---
layout: post
title:  "AWS Lambda-Java Script & Node.js 基础 "
date:   2017-09-22 14:00:05
categories: IoT
excerpt: AWS Lambda Dev Env，JavaScript & Node.js 基本知识。
---

* content
{:toc}


---

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


### JavaScript基础

#### 2.1	JavaScript数据类型

- 基本数据类型：number, boolean, string, null及undefined
- 复杂数据类型：array, function, 及object。访问复杂类型，访问的是对值的引用。
	- 例如： var a = [‘hello’, ‘world’]; b = a; b[0]=‘bye’; 那么 a[0] = ‘bye’。
- JavaScript同其他面向对象的语言一样有相应的构造器，很难判断数据的类型。

		var a = ‘hello’; 
		var b = new String(‘world’);
		typeof a == ‘string’ 
		typeof b == ‘object’;
		a instanceof String == false;
		b instanceof String == true;


#### 2.2	函数

- 函数可以作为引用存储在变量中，随后像其他对象一样进行传递：var a = function(a,b){}; console.log(a(1,2));
- 使用.call和.apply方法可以改变函数执行时的上下文(this)。例如：
		
		var test = {
			a : 1,
		    b : 2,
		    sum : function (a,b) {
		        var self = this;
		        function getA() {
		            return self.a;
		        }
		        function getB(){
		            return self.b;
		        }
		        console.log(a);
		        console.log(b);
		        return getA() + getB();
		    }
		}
		var obj = {a:3,b:4};
		console.log(test.sum.call(obj,5,6)); 		// 5, 6, 7
		console.log(test.sum.apply(obj,[5,6])); 	//5, 6, 7
- 函数的属性--参数数量。例如：var a = function(a, b, c); a.length == 3;// true

#### 2.3	类

- JavaScript中没有class关键词，只能通过函数来定义: function Animal() {};
- 通过prototype属性给所有Animal的实例定义函数：Animal.prototype.eat = function(food){//eat method};
- 定义继承链：function Dog() {}； Dog.prototype = new Animal(); 
- 为子类定义属性：Dog.prototype.type = 'domestic';
- 子类通过prototype重写和调用父类函数：Dog.prototype.eat = function(food){ Animal.prototype.eat.call(this, food); }

#### 2.4	V8中的JavaScript
V8是Google发布的开源JavaScript引擎，采用 C++ 编写，被应用在Google的Chrome浏览器中。

- 遍历数组方法：[1,2,3].forEach(function(v) { console.log(v); });
- 过滤数组方法：[1,2,3].filter(function(v) { return v<3; }); //返回[1, 2]
- 改变数组每个元素的值: [1,2,3].map(function(v){ return v*2; }); //返回[2, 4, 6]
- 字符串方法： '  hello  '.trim(); //"hello"
- 非标准的函数属性名: var a = function eat(){}； a.name == 'eat'; //true
- _PROTO_ 继承： 使得定义继承链变得更加容易

		function Animal(){};
		function Dog(){};
		Dog.prototype._proto_ = Animal.prototype;
- 存取器：__defineGetter__, __defineSetter__

		Dog.prototype.__defineGetter__('eat', function(){return 'bone';});
		var dog = new Dog();
		dog.eat; // bone

### Node.js基础

#### 3.1	Node.js阻塞与非阻塞IO

- Node.js是基于Chrome V8引擎的JavaScript运行环境。内部采用一个长期运行的进程和事件轮询机制。
- V8首次调用一个函数时，会创建一个调用堆栈，或者称为执行堆栈。如果一个函数又去调用另一个函数的话，v8就会将它添加到堆栈上。
假设某个函数是为了获取数据库的数据，调用堆栈中就只有数据库调用。由于调用是非阻塞的，当数据库IO完成时，就取决于事件轮询何时再初始化新的调用堆栈。

#### 3.2	Node中的JavaScript

Node.js除了提供和浏览器一样的基本语言外，还提供了构建强大网络应用所需的API。

- 全局对象：console.log/error; process.nextTick(function(){});
- 模块系统：Node摒弃了采用一堆全局变量，转而引入了模块系统，模块系统有三个核心的全局对象require, module, exports
- 绝对和相对模块:
	- 绝对模块：Node通过其内部node_modules查找到的模块，或者Node内置的如fs这样的模块。可直接通过require('fs')
	- 相对模块：将require指向一个相对工作目录中的JavaScript文件。
	- 要让模块暴露一个API成为require调用的返回值，需要依靠module和exports.
- exports和module.exports区别：

		module.exports = Person;
		function Person(name) {
			this.name = name;
		};
		Person.prototype.speak = function(){
		};
		var Person = require('./person.js');
		var Jack = new Person('jack');
		Jack.speak(); 

	- 每个node.js文件都会创建一个module对象，同时module对象会创建一个exports的属性.module.exports = {}
	- exports和module.exports指向同一块内存，require返回的值是module.exports而不是exports.
	
- 事件：Node.js所有的异步I/O操作在完成时都会发送一个事件到事件队列。所有这些产生事件的对象都是events.EventEmitter的实例。
				
		var EventEmitter = require('events').EventEmitter; 
		var event = new EventEmitter(); 
		event.on('my_event', function() { 
		    console.log('some_event 事件触发'); 
		}); 
		setTimeout(function() { 
		    event.emit('my_event'); 
		}, 1000);
- 使自己的类具备事件处理功能：

		var EventEmitter = require('events').EventEmitter; 
		MyClass = function(){};
		MyClass.prototype.__proto__ = EventEmitter.prototype;
		var myClass = new MyClass;
		myClass.on('some_event', function() { 
		    console.log('事件触发'); 
		}); 
		setTimeout(function() { 
		    myClass.emit('some_event'); 
		}, 1000);

- Buffer：除了模块之外，Node还弥补了语言另外一个不足，那就是二进制数据的处理。

#### 3.3	NPM使用

NPM是随同Node.js一起安装的包管理工具，主要用来给NPM服务器上传或者下载第三方包。最常见的依赖包可以参照<https://www.npmjs.com/browse/depended>。

- npm的包安装分为本地安装（local）、全局安装（global）npm install express (-g)
- 查看安装信息：npm list -g
- 查看某个模块的版本号：npm list express
- 使用package.json:位于模块的目录下，用于定义包的属性。可通过npm init来创建。 我们可以在script里面配置一些常用的命令比如：start/stop/restart等

		{
		  	"name": "webhook",
		  	"version": "1.0.0",
		  	"description": "sample webhook app",
		  	"main": "server.js",
		  	"scripts": {
		    	"test": "echo \"Error: no test specified\" && exit 1",
		    	"start": "node server.js"
		  	},
		  	"author": "Liang",
		  	"license": "ISC",
		  	"dependencies": {
		    	"bluebird": "^3.5.0",
			    "body-parser": "^1.17.2",
			    "config": "^1.26.2",
			    "express": "^4.15.4",
			    "fs": "0.0.1-security",
			    "prettyjson": "^1.2.1",
			    "http-signature": "^1.1.1",
			    "request": "^2.81.0",
			    "moment": "~2.18.1"
		  	}
		}

至此，基本的Java Script语法和Node.js的基本概念已基本描述清楚，会单独一篇博文来总结Lambda App的开发。