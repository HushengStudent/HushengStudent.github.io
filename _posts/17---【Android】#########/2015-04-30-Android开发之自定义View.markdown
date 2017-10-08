---
layout: post
title:  "[Android]Android开发之自定义View"
date:   2015-04-30 12:10:10
categories: 17---【Android】#########
comments: true
---
Android系统的视图结构的设计也采用了组合模式，即View作为所有图形的基类，Viewgroup对View继承扩展为视图容器类，由此就得到了视图部分的基本结构--树形结构。

![图片1](http://owk5gjdrg.bkt.clouddn.com/0004Android%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89View.png)

### View定义了绘图的基本操作

基本操作由三个函数完成：measure()、layout()、draw()，其内部又分别包含了onMeasure()、onLayout()、onDraw()三个子方法。具体操作如下：
#### 1、measure操作
measure操作主要用于计算视图的大小，即视图的宽度和长度。在view中定义为final类型，要求子类不能修改。measure()函数中又会调用下面的函数：

（1）onMeasure()，视图大小的将在这里最终确定，也就是说measure只是对onMeasure的一个包装，子类可以覆写onMeasure()方法实现自己的计算视图大小的方式，并通过setMeasuredDimension(width, height)保存计算结果。
#### 2、layout操作
layout操作用于设置视图在屏幕中显示的位置。在view中定义为final类型，要求子类不能修改。layout()函数中有两个基本操作：

（1）setFrame（l,t,r,b），l,t,r,b即子视图在父视图中的具体位置，该函数用于将这些参数保存起来；
		

（2）onLayout()，在View中这个函数什么都不会做，提供该函数主要是为viewGroup类型布局子视图用的；
#### 3、draw操作
draw操作利用前两部得到的参数，将视图显示在屏幕上，到这里也就完成了整个的视图绘制工作。子类也不应该修改该方法，因为其内部定义了绘图的基本操作：

（1）绘制背景；
		
（2）如果要视图显示渐变框，这里会做一些准备工作；
		
（3）绘制视图本身，即调用onDraw()函数。在view中onDraw()是个空函数，也就是说具体的视图都要覆写该函数来实现自己的显示（比如TextView在这里实现了绘制文字的过程）。而对于ViewGroup则不需要实现该函数，因为作为容器是“没有内容“的，其包含了多个子view，而子View已经实现了自己的绘制方法，因此只需要告诉子view绘制自己就可以了，也就是下面的dispatchDraw()方法;
		
（4）绘制子视图，即dispatchDraw()函数。在view中这是个空函数，具体的视图不需要实现该方法，它是专门为容器类准备的，也就是容器类必须实现该方法；
		
（5）如果需要（应用程序调用了setVerticalFadingEdge或者setHorizontalFadingEdge），开始绘制渐变框；
		
（6）绘制滚动条；
		
从上面可以看出自定义View需要最少覆写onMeasure()和onDraw()两个方法。
	  
### ViewGroup中的扩展操作：
首先Viewgroup是一个抽象类。
#### 1、对子视图的measure过程
（1）measureChildren()，内部使用一个for循环对子视图进行遍历，分别调用子视图的measure()方法；
		
（2）measureChild()，为指定的子视图measure，会被 measureChildren调用；
		
（3）measureChildWithMargins()，为指定子视图考虑了margin和padding的measure；
		
以上三个方法是ViewGroup提供的3个对子view进行测量的参考方法，设计者需要在实际中首先覆写onMeasure()，之后再对子view进行遍历measure，这时候就可以使用以上三个方法，当然也可以自定义方法进行遍历。
#### 2、对子视图的layout过程
在ViewGroup中onLayout()被定义为abstract类型，也就是具体的容器必须实现此方法来安排子视图的布局位置，实现中主要考虑的是视图的大小及视图间的相对位置关系，如gravity、layout_gravity。
#### 3、对子视图的draw过程
（1）dispatchDraw()，该方法用于对子视图进行遍历然后分别让子视图分别draw，方法内部会首先处理布局动画（也就是说布局动画是在这里处理的），如果有布局动画则会为每个子视图产生一个绘制时间，之后再有一个for循环对子视图进行遍历，来调用子视图的draw方法（实际为下边的drawChild()）；

（2）drawChild()，该方法用于具体调用子视图的draw方法，内部首先会处理视图动画（也就是说视图动画是在这里处理的），之后调用子视图的draw()。

从上面分析可以看出自定义viewGroup的时候需要最少覆写onMeasure()和onLayout()方法，其中onMeasure方法中可以直接调用measureChildren等已有的方法，而onLayout方法就需要设计者进行完整的定义；一般不需要覆写以dispatchDraw()和drawChild()这两个方法，因为上面两个方法已经完成了基本的事情。

自定义View首先要实现一个继承自View的类。添加类的构造方法，override父类的方法，如onDraw，（onMeasure）等。如果自定义的View有自己的属性，需要在values下建立attrs.xml文件，在其中定义属性，同时代码也要做修改。

![图片1](http://owk5gjdrg.bkt.clouddn.com/0005Android%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89View.png)