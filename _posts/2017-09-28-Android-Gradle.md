---
layout: post
title:  "理解Android之Gradle"
date:   2017-05-15 19:00:05
categories: Android
excerpt: Android APP 开发实践笔记。
---

* content
{:toc}

在Gradle爆红之前，常用的构建工具是ANT，然后又进化到Maven。但是二者都有一些缺点。比如，Maven编译规则是用XML来编写的。XML虽然通俗易懂，但是很难在xml中描述if{某条件成立，编译某文件}/else{编译其他文件}这样有不同条件的任务。Gradle选择了Groovy。 


---

### Groovy介绍

Groovy是一种动态语言。它和Java一样，也运行于Java虚拟机中。当执行Groovy脚本时，Groovy会先将其编译成Java类字节码，然后通过Jvm来执行这个Java类。  

---

#### Groovy开发环境

在Ubuntu的的开发环境中：

- curl -s "https://get.sdkman.io" | bash
- source ~/.bashrc
- gvm install groovy
 
创建test.groovy文件，执行groovy test.groovy

#### 基本知识  
1. Groovy的注释和Java一样
2. Groovy的语句可以不用分号结尾
3. Groovy支持动态类型，即定义的时候不需要指定类型。变量定义可以使用关键字def
4. 函数定义时，参数的类型可以不用指定，并且函数的返回值也可以是无类型的，函数中最后一行的执行结果就是函数的返回值
5. Groovy对字符串的支持非常强大
	
		a = 'I am $x dollar' //单引号''中的内容严格对应Java中的String，不对$符号进行转义
		b = "I am $x dollar" //双引号""的内容则和脚本语言的处理有点像，它会对$表达式先求值,x = 1;
		c = ''' s
			ass
			ass ''' 		 //三个引号'''xxx'''中的字符串支持随意换行
		
#### Groovy基本数据类型  

1. 基本数据类型:	
	- 作为动态语言，Groovy世界中的所有事物都是对象。比如int对应为Integer，boolean对应为Boolean。
2. Groovy中的容器类
	- List:底层对应Java中的List接口，一般用ArrayList作为真正的实现类
	
			def aList = [5,'string',true] //List由[]定义，其元素可以是任何对象 
			aList[100] = 100			  //List会自动往该索引添加元素  
			println(aList.size)  		  //结果是101

	- Map：底层对应Java中的LinkedHashMap

			def map = [3:'c',2:'b',1:'a',5:'e',4:'d']
			map.name="Lucy"  //设置集合的内容，其中name为key  
			map.put("sex","女")  //向map中添加元素  
			println map.get("name")  //取得集合的单个内容
			map.each { it ->  
				println it.key+","+it.value  
			}  
	- Range:List的拓展,用..表示范围操作符，用来指定左右边界
			
			assert (0..10).contains(5) == true
			assert(0..<10).contains(10) == false
			def r1 = 1..6
			def r2 = 1..<6
			println(r1.from)
			println(r2.to)
			def str = ''
			(9..5).each{element->
			    str+=element
			}
			println(str)

3. 闭包  
	- 闭包(Closure)，是Groovy中非常重要的一个数据类型,它代表了一段可执行的代码。其外形如下：

			def aClosure = {				 //闭包是一段代码，所以需要用花括号括起来..  
    			String param1, int param2 ->  //箭头前面是参数定义，箭头后面是代码  
    			println"this is code" 		 //代码，最后一句是返回值 
			}  
	- 闭包定义好以后，调用方法就是 闭包对象.call(参数) 或者 闭包对象(参数)。例如：

			def greeting = { it -> "Hello, $it!"} //闭包隐含一个参数it，和this作用类似 
			println greeting('world')
 	- 可以将闭包作为参数传入另外一个闭包：

			def myClosure = {a1, b1 -> 
				a1 + b1
			}
			def testClosure = { a, b, c ->  	
				println a+", "+b+"," +c(a, b)
			}
			testClosure (4, "test", myClosure)
	- 闭包的快捷写法：

			def runTwiceClosures = { c -> c() + c() } 
			println runTwiceClosures( { 'plate' } )
			println runTwiceClosures(){ 'bowl' } //闭包作为闭包或方法的最后一个参数。可将闭包从参数圆括号中提取出来接在最后
			println runTwiceClosures{ 'mug' }    //闭包是唯一的一个参数，闭包或方法参数所在的圆括号也可以省略，比如each			
	- 闭包可以通过定义最后一个参数声明为Object[]来获取任意多个参数：
	
			def c = { arg, Object[] extras ->  
			  def list= []  
			  list<< arg  
			  extras.each{ list<< it }  
			  list.each{
				println it
			  }  
			}
			c('a', 1, "aaaa")
	- 如果闭包的参数中没有list,那么传入参数可以设置为list,里面的参数将分别传入闭包参数：
	
			def c = { a, b, c -> 
				println a+b+c
			}
			def list = [1, 2, 3]
			c(list)

### Gradle
Gradle是一个编程框架，

---



---
