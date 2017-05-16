---
layout: post
title:  "Android资源文件使用"
date:   2017-05-15 19:00:05
categories: Android
excerpt: Android APP GUI开发实践笔记。
---

* content
{:toc}

Android应用程序会使用各种资源，比如图片，字串，动画等，Android支持使用XML文件来定义这些资源及UI布局。  
![resource布局](http://itliang.github.io/blog/public/img/resources.png)

---

### Android颜色渲染

利用ProterBuff.Mode我们可以完成任意2D图像测操作。从下面的图中可以看出ProterBuff.Mode为枚举类，一共有16种枚举值：  
![resource布局](http://itliang.github.io/blog/public/img/color.png)

1. PorterDuff.Mode.CLEAR：所绘制不会提交到画布上
2. PorterDuff.Mode.SRC：显示上层绘制图片
3. PorterDuff.Mode.DST：显示下层绘制图片
4. PorterDuff.Mode.SRC_OVER：正常绘制显示，上下层绘制叠盖
5. PorterDuff.Mode.DST_OVER：上下层都显示，下层居上显示
6. PorterDuff.Mode.SRC_IN：取两层绘制交集，显示上层
7. PorterDuff.Mode.DST_IN：取两层绘制交集，显示下层
8. PorterDuff.Mode.SRC_OUT：取上层绘制非交集部分
9. PorterDuff.Mode.DST_OUT：取下层绘制非交集部分
10. PorterDuff.Mode.SRC_ATOP：取下层非交集部分与上层交集部分
11. PorterDuff.Mode.DST_ATOP：取上层非交集部分与下层交集部分
12. PorterDuff.Mode.XOR：异或,去除两图层交集部分
13. PorterDuff.Mode.DARKEN：取两图层全部区域，交集部分颜色加深
14. PorterDuff.Mode.LIGHTEN：取两图层全部，点亮交集部分颜色
15. PorterDuff.Mode.MULTIPLY：取两图层交集部分叠加后颜色 
16. PorterDuff.Mode.SCREEN：取两图层全部区域，交集部分变为透明色
    
android:background(直接修改控件的背景颜色)  
android:backgroundTint(将设置的颜色和原来的背景进行一个叠加的过程,如何叠加由mode决定)  
android:backgroundTintMode(这个属性传的值就是刚刚上面那些PorterDuff.Mode中的值)  

---

### Selector使用

Selector主要用来改变ListView和Button控件的默认背景。它分为两种，Color-Selector 和Drawable-Selector。  

#### Color-Selector  
color-selector是颜色状态列表，可以跟color一样使用，文件一般置于/res/color/filename.xml。语法如下:
  
	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
		<item
	        android:color="hex_color"               //颜色值，
	        android:state_pressed=["true" | "false"]//是否触摸 
	        android:state_focused=["true" | "false"]//是否获得焦点
	        android:state_selected=["true" | "false"]//是否被状态
	        android:state_checkable=["true" | "false"]//是否可选
	        android:state_checked=["true" | "false"]//是否选中
	        android:state_enabled=["true" | "false"]//是否可用
	        android:state_window_focused=["true" | "false"] />//是否窗口聚焦
	</selector>


#### Drawable-Selector  
drawable-selector是背景图状态列表，可以跟图片一样使用，背景会根据组件的状态变化而变化。文件存储于/res/drawable/filename.xml。语法如下：  

	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android"
    	android:constantSize=["true" | "false"] //drawable的大小是否当中状态变化，true表示是变化，false表示不变换，默认为false
    	android:dither=["true" | "false"] //当位图与屏幕的像素配置不一样时(例如，一个ARGB为8888的位图与RGB为555的屏幕)会自行递色(dither)。设置为false时不可递色。默认true
    	android:variablePadding=["true" | "false"] >//内边距是否变化，默认false
    	<item
	        android:drawable="@[package:]drawable/drawable_resource"//图片资源
	        android:state_pressed=["true" | "false"]//是否触摸
	        android:state_focused=["true" | "false"]//是否获取到焦点
	        android:state_hovered=["true" | "false"]//光标是否经过
	        android:state_selected=["true" | "false"]//是否选中
	        android:state_checkable=["true" | "false"]//是否可勾选
	        android:state_checked=["true" | "false"]//是否勾选
	        android:state_enabled=["true" | "false"]//是否可用
	        android:state_activated=["true" | "false"]//是否激活
	        android:state_window_focused=["true" | "false"] />//所在窗口是否获取焦点
	</selector>

---

### Shape使用

Shape在美化控件中的作用至关重要。定义文件位于res/drawable/filename.xml 语法如下：
	
	<?xml version="1.0" encoding="utf-8"?>
	<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    	<corners android:radius="14dp" />
    	<solid android:color="@color/… " />
	</shape>  

其中具体的属性参照下面注释：  
	
	<shape android:shape=["rectangle" | "oval" | "line" | "ring"]>  
		<gradient	//渐变
			android:startColor	起始颜色
			android:endColor	结束颜色
			android:angle	渐变角度，0从左到右，90表示从下到上，数值为45的整数倍，默认为0；
			android:type	渐变的样式 liner线性渐变 radial环形渐变 sweep	/>
		<solid	//内部填充 
			android:color //填充的颜色 	/>
		<stroke //描边
			android:width	描边的宽度
			android:color 	描边的颜色
			android:dashWidth	表示'-'横线的宽度
			android:dashGap 	表示'-'横线之间的距离 		/>
		<corners //圆角
			android:radius  
			android:topRightRadius  
			android:bottomLeftRadius 
			android:topLeftRadius 
			android:bottomRightRadius 	/>
		<padding //边界填充
			android:bottom="1.0dip"	底部填充
			android:left="1.0dip" 	左边填充
			android:right="1.0dip"	右边填充
			android:top="0.0dip"	上面填充  />
	</shape>

---
