# article
some article about programm.
***

##20160116

>For one kiss,I would defy a thousand Wessexes

###使用贝塞尔曲线制作可伸缩视图动画(译文)
原文链接[Elastic view with animation using UIBezierPath](http://iostuts.io/2015/10/17/elastic-bounce-using-uibezierpath-and-pan-gesture/)

###最终效果!
该篇教程的最终效果以及[代码](https://github.com/gontovnik/DGElasticPullToRefresh)
![效果图](http://iostuts.io/content/images/2015/10/DGElasticPullToRefresh1.gif)

###教程的开发要求

* Xocde7
* swfit2
* 贝塞尔曲线以及滑动手势事件的基本知识以及理解

###理解逻辑
就像你的大概理解一样，我们主要是使用贝塞尔曲线的技巧来实现。我们先通过贝塞尔曲线创建一个CAShapeLayer。贝塞尔曲线上面有7个控制点，通过手指的滑动来改变控制点的位置，时刻改变CAShapeLayer的形状。每个控制点用一个可见的视图来代表着。下面图片上的红色点就是控制点的具体位置。为了达到该目的，我们在工程中使用iCADDispalyLink定时器来在主线程RunLoop的每一帧中执行更新贝塞尔曲线。	

![初始位置](http://iostuts.io/content/images/2015/10/ControlPoints1.png)
![拉伸过程中](http://iostuts.io/content/images/2015/10/ControlPoints2.png)

当我们释放手指之后，图层就会以一个弹性动画回到初始位置。为了可以给图层添加动画，我们需要及时更新贝塞尔曲线

###开始编写代码
建立一个工程并粘贴下面的代码到类的声明中


			private let minimalHeight: CGFloat = 50.0  
			private let shapeLayer = CAShapeLayer()
		
		// MARK: -
		
			override func loadView() {  
		    	super.loadView()

		    shapeLayer.frame = CGRect(x: 0.0, y: 0.0, width: view.bounds.width, height: minimalHeight)
		    shapeLayer.backgroundColor = UIColor(red: 57/255.0, green: 67/255.0, blue: 89/255.0, alpha: 1.0).CGColor
		    view.layer.addSublayer(shapeLayer)
		
		    view.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: "panGestureDidMove:"))
			}
		
			func panGestureDidMove(gesture: UIPanGestureRecognizer) {  
		    		if gesture.state == .Ended || gesture.state == .Failed || gesture.state == .Cancelled {
		
		    	} else {
		        	shapeLayer.frame.size.height = minimalHeight + max(gesture.translationInView(view).y, 0)
		    	}
		    }
		
			override func preferredStatusBarStyle() -> UIStatusBarStyle {  
		    return .LightContent
	}

上面代码做了些什么?
* 定义了两个变量，**shapeLayer**用来结合贝塞尔曲线的显示层。**minimalHeight**用来表示**shapeLayer**的最小高度 
* 添加 一个 shape layer到 主视图的view
* 为主视图添加滑动手势
* 为屏幕的滑动添加一个手势事件的方法**panGestureDidMove**，用来时刻改变着shape layer的高度
* 重写preferredStatusBarStyle方法来定义状态栏的颜色变成白色。

接下来run一下我们的工程，最终效果就是这样

![效果](http://iostuts.io/content/images/2015/10/Builds1.gif)

它的确跟我们想要的那样运行，但是除了一样东西。图层高度的变化伴随着一个延时动画，出现的原因是因为图层的隐式动画。我们可以直接关闭这些隐式动画。只需要将下面的代码添加在shape layer添加到主视图的图层之前即可关闭关于position，bounds和path的隐式动画。

	shapeLayer.actions = ["position" : NSNull(), "bounds" : NSNull(), "path" : NSNull()]  
	
重新run一下工程
![](http://iostuts.io/content/images/2015/10/Builds2.gif)

接下来我们需要做的是就是添加控制点视图(L3, L2, L1, C, R1, R2, R3,).OK，我们一步一步来。

 *定义波峰的最大高度变量：maxWaveHeight

	private let maxWaveHeight: CGFloat = 100.0 
	
我们定义并使用该变量的唯一原因就是为了使得波形变得更好看。如果我们没有定义波峰的最大高度，那么波形就会变得太大而且太难看。

 * 定义下面这些控制点的视图

	private let l3ControlPointView = UIView()  
	private let l2ControlPointView = UIView()  
	private let l1ControlPointView = UIView()  
	private let cControlPointView = UIView()  
	private let r1ControlPointView = UIView()  
	private let r2ControlPointView = UIView()  
	private let r3ControlPointView = UIView()
	
* 定义这些控制点的视图大小以及颜色(比如使用红色，是为了在前期调试可以更好容易辨识，在教程的最后，我们就隐藏这些控制点)。并在**loadView()**方法中添加下面这些代码

	l3ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	l2ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	l1ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	cControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	r1ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	r2ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)  
	r3ControlPointView.frame = CGRect(x: 0.0, y: 0.0, width: 3.0, height: 3.0)
	
	l3ControlPointView.backgroundColor = .redColor()  
	l2ControlPointView.backgroundColor = .redColor()  
	l1ControlPointView.backgroundColor = .redColor()  
	cControlPointView.backgroundColor = .redColor()  
	r1ControlPointView.backgroundColor = .redColor()  
	r2ControlPointView.backgroundColor = .redColor()  
	r3ControlPointView.backgroundColor = .redColor()
	
	view.addSubview(l3ControlPointView)  
	view.addSubview(l2ControlPointView)  
	view.addSubview(l1ControlPointView)  
	view.addSubview(cControlPointView)  
	view.addSubview(r1ControlPointView)  
	view.addSubview(r2ControlPointView)  
	view.addSubview(r3ControlPointView) 


* 添加一个UIView的extension到ViewController的视图之前
	
	extension UIView {  
	    func dg_center(usePresentationLayerIfPossible: Bool) -> CGPoint {
	    if usePresentationLayerIfPossible, let presentationLayer = layer.presentationLayer() as? CALayer 	{
	            return presentationLayer.position
	        }
	        return center
	    }
	}

当你需要对一个UIView的位置改变添加动画时候，你需要访问这个view的frame。那么UIView.center方法会给予你view的最终位置的frame而不是当前值。为了达到这个目的我们将创建一个extension用来在我们需要的时候获取UIView.layer.presentationLayer的位置。关于presentationLayer的相关资料可以点击[这里](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CALayer_class/#//apple_ref/occ/instm/CALayer/presentationLayer)


* 定义 currentPath 方法

	    private func currentPath() -> CGPath {  
	    let width = view.bounds.width
	
	    let bezierPath = UIBezierPath()
	
	    bezierPath.moveToPoint(CGPoint(x: 0.0, y: 0.0))
	    bezierPath.addLineToPoint(CGPoint(x: 0.0, y: l3ControlPointView.dg_center(false).y))
	    bezierPath.addCurveToPoint(l1ControlPointView.dg_center(false), controlPoint1: l3ControlPointView.dg_center(false), controlPoint2: l2ControlPointView.dg_center(false))
	    bezierPath.addCurveToPoint(r1ControlPointView.dg_center(false), controlPoint1: cControlPointView.dg_center(false), controlPoint2: r1ControlPointView.dg_center(false))
	    bezierPath.addCurveToPoint(r3ControlPointView.dg_center(false), controlPoint1: r1ControlPointView.dg_center(false), controlPoint2: r2ControlPointView.dg_center(false))
	    bezierPath.addLineToPoint(CGPoint(x: width, y: 0.0))
	
	    bezierPath.closePath()
	
	    return bezierPath.CGPath
	}

这个方法是用来返回当前的shape layer的CGPath。它的形状是依赖于控制点的位置。

* 定义updateShapeLayer 方法

	func updateShapeLayer() {  
    	shapeLayer.path = currentPath()
	}
	
这个方法用来更新shape layer。它不是一个私有的方法，因为它将被CADisplayLink 定时器添加 Seletor() 方法。

* 定义 layoutControlPoints 方法

		private func layoutControlPoints(baseHeight baseHeight: CGFloat, waveHeight: CGFloat, locationX: CGFloat) {  
	    let width = view.bounds.width
	
	    let minLeftX = min((locationX - width / 2.0) * 0.28, 0.0)
	    let maxRightX = max(width + (locationX - width / 2.0) * 0.28, width)
	
	    let leftPartWidth = locationX - minLeftX
	    let rightPartWidth = maxRightX - locationX
	
	    l3ControlPointView.center = CGPoint(x: minLeftX, y: baseHeight)
	    l2ControlPointView.center = CGPoint(x: minLeftX + leftPartWidth * 0.44, y: baseHeight)
	    l1ControlPointView.center = CGPoint(x: minLeftX + leftPartWidth * 0.71, y: baseHeight + waveHeight * 0.64)
	    cControlPointView.center = CGPoint(x: locationX , y: baseHeight + waveHeight * 1.36)
	    r1ControlPointView.center = CGPoint(x: maxRightX - rightPartWidth * 0.71, y: baseHeight + waveHeight * 0.64)
	    r2ControlPointView.center = CGPoint(x: maxRightX - (rightPartWidth * 0.44), y: baseHeight)
	    r3ControlPointView.center = CGPoint(x: maxRightX, y: baseHeight)
		}

在这里我们需要对方法内的定义的变量做出一些解释
	1 **baseHeight** - 这是一个基础的layer高度。baseHeight + waveHeight = 整个shape layer的高度
	2 **waveHeight** - 曲线的波形高度。这个高度不能超过之前定义的最大波形高度**maxWaveHeight**。否则波形将会很难看
	3 **locationX** - 在屏幕上手指触摸点的**X**坐标
	4 **width** - 主视图的宽度
	5  **mineLeftX** - 定义**l3ControlPointView**最小**x**坐标位置。这个变量的值可以少于0。因此shape layer在视觉上看起来更加好以及清晰。
	6 **maxRightX** - 跟**mineLeftX**的作用相当
	7 **leftPartWidth** - 定义**mineLeftX** 和 **locationX**之间的距离
	8 **rightPartWidth** - 定义**locationX** 与 **maxRightX**之间的距离
	
在这里，你可能会问控制点的位置为什么使用这些参数值。答案其实很简单:我使用**paintCode**
来经过多次模拟贝塞尔曲线来得到的。当我发现我需要的这些值之后，我就将他们放到代码中并多次模拟曲线直到获取最佳点的值。

* 接下来更新 **panGestureDidMove**方法，因此控制点的位置将跟随这手指触摸点的位置。用下面的代码覆盖原来方法的代码

		func panGestureDidMove(gesture: UIPanGestureRecognizer) {  
	   	 if gesture.state == .Ended || gesture.state == .Failed || gesture.state == .Cancelled {
	
	   	 } else {
	      	  let additionalHeight = max(gesture.translationInView(view).y, 0)
	
	        let waveHeight = min(additionalHeight * 0.6, maxWaveHeight)
	        let baseHeight = minimalHeight + additionalHeight - waveHeight
	
	        let locationX = gesture.locationInView(gesture.view).x
	
	        layoutControlPoints(baseHeight: baseHeight, waveHeight: waveHeight, locationX: locationX)
	        updateShapeLayer()
	    }
		}

我们需要做的是计算波形的高度，基本高度，手指的位置以及调用**layoutControlPoints**方法重新来设置控制点的位置并调用**updateShapeLayer**方法及时更新shape layer的形状。

*添加这两行代码到**loadView()**方法的结尾处。因此在我们打开该APP之后我们就先更新一次shape layer。

	layoutControlPoints(baseHeight: minimalHeight, waveHeight: 0.0, locationX: view.bounds.width / 2.0)  
	updateShapeLayer()  

* 改变shape layer的backgroundColor:

		shapeLayer.backgroundColor = UIColor(red: 57/255.0, green: 67/255.0, blue: 89/255.0, alpha: 1.0).CGColor

* 改变填充颜色

		shapeLayer.fillColor = UIColor(red: 57/255.0, green: 67/255.0, blue: 89/255.0, alpha: 1.0).CGColor


重新run一次工程并得到下面的效果
	
![](http://iostuts.io/content/images/2015/10/Builds3.gif)


剩下最后一件事就是当我们释放手指之后添加弹动动画

OK，我们一步一步来完成这些。

* 定义 **displayLink** 变量

		private var displayLink: CADisplayLink!

* 然后在**loadView()**方法中初始化这个定时器

		displayLink = CADisplayLink(target: self, selector: Selector("updateShapeLayer"))  
		displayLink.addToRunLoop(NSRunLoop.mainRunLoop(), forMode: NSDefaultRunLoopMode)  
		displayLink.paused = true

就像教程前面提到一样。我们的在系统RunLoop，更新每一帧就会通过CADisplayLink来调用**updateShapeLayer**方法来及时更新shape layer 形状

* 定义 **animating** 变量

			private var animating = false {  
		    didSet {
		        view.userInteractionEnabled = !animating
		        displayLink.paused = !animating
		    }
		}

它用来打开或者关闭主视图的交互功能以及displayLink定时器。

* 更新 **currentPath** 方法。因此**dg_center(Bool)**在调用的时候会使用上面定义的**animating**变量

			private func currentPath() -> CGPath {  
		    let width = view.bounds.width
		
		    let bezierPath = UIBezierPath()
		
		    bezierPath.moveToPoint(CGPoint(x: 0.0, y: 0.0))
		    bezierPath.addLineToPoint(CGPoint(x: 0.0, y: l3ControlPointView.dg_center(animating).y))
		    bezierPath.addCurveToPoint(l1ControlPointView.dg_center(animating), controlPoint1: l3ControlPointView.dg_center(animating), controlPoint2: l2ControlPointView.dg_center(animating))
		    bezierPath.addCurveToPoint(r1ControlPointView.dg_center(animating), controlPoint1: cControlPointView.dg_center(animating), controlPoint2: r1ControlPointView.dg_center(animating))
		    bezierPath.addCurveToPoint(r3ControlPointView.dg_center(animating), controlPoint1: r1ControlPointView.dg_center(animating), controlPoint2: r2ControlPointView.dg_center(animating))
		    bezierPath.addLineToPoint(CGPoint(x: width, y: 0.0))
		
		    bezierPath.closePath()
		
		    return bezierPath.CGPath
		}

* 最后一步就是更新**panGestureDidMove**方法。用下面的代码覆盖之前的代码

			if gesture.state == .Ended || gesture.state == .Failed || gesture.state == .Cancelled {  
		    let centerY = minimalHeight
		
		    animating = true
		    UIView.animateWithDuration(0.9, delay: 0.0, usingSpringWithDamping: 0.57, initialSpringVelocity: 0.0, options: [], animations: { () -> Void in
		        self.l3ControlPointView.center.y = centerY
		        self.l2ControlPointView.center.y = centerY
		        self.l1ControlPointView.center.y = centerY
		        self.cControlPointView.center.y = centerY
		        self.r1ControlPointView.center.y = centerY
		        self.r2ControlPointView.center.y = centerY
		        self.r3ControlPointView.center.y = centerY
		        }, completion: { _ in
		            self.animating = false
		    })
		} else {
		    let additionalHeight = max(gesture.translationInView(view).y, 0)
		
		    let waveHeight = min(additionalHeight * 0.6, maxWaveHeight)
		    let baseHeight = minimalHeight + additionalHeight - waveHeight
		
		    let locationX = gesture.locationInView(gesture.view).x
		
		    layoutControlPoints(baseHeight: baseHeight, waveHeight: waveHeight, locationX: locationX)
		    updateShapeLayer()
		}

在方法中我们添加了一个弹簧动画到每一个控制点视图，在它每次返回到初始化位置的过程中都得到了非常漂亮的过渡过程。通过不停地调试这些动画参数，或者你可以得到一个更加漂亮的的动画效果。

重新run一下工程就会得到下面amazing效果：
	![](http://iostuts.io/content/images/2015/10/Builds4.gif)
	

这次最后的最后就是将每个控制点的位置backgroundColor为透明颜色即可。

![](http://iostuts.io/content/images/2015/10/Builds5.gif)


####OK，完成到此结束。enjoy！

---


##2016-01-11

>I want to be a model

### YYModel粗读

####前言
记得在刚开始学习做项目的时候，就开始使用**MJExtension**,就感受到它的便利与快捷。就想着知道它是如何实现，当时也是看了一下源码，发现原来是用**runtime**来实现。想着自己那时候能力和时间还不够，就没有深入学习它的源码。直到最近**YYKit**开源，在微博上面引起巨大反应。看了作者GitHub的项目，除了钦佩作者iOS功力高深，还更加突出自己的差距而需要继续努力学习。因此，就先找了组件之一的**YYModel**的源码进行了学习。


##2016-01-06

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipayTitle.jpg)

> no pay , no gain.

###支付宝集成

####前言
支付宝SDK的集成还是很简单的，之前觉得唯一的坎就是真正的商家key可以用,用之来验证真正的支付过程。最近刚好要搞支付，这下子爽了。下面就是集成步骤。

 * 先到支付宝文档中心下载[SDK以及对应的demo](https://doc.open.alipay.com/doc2/detail?treeId=59&articleId=103563&docType=1)来看一下。先run一下demo感受一下，看一下工程的文件结构。这时候应该就有了一定的了解。
 * 新建一个工程，然后打开SDK的demo工程所在的文件夹。把下面几个文件复制并放到新工程目录下的文件夹。
  ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDK.png)
 * 在新工程中添加AlipaySDK文件夹的全部文件
 * 在新工程中的Target-->Build setting-->搜索“Header search path”,填入AlipaySDK.framework在工程中的相对路径。
   ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDKpath.png)
 * 在Target-->info-->URL Types-->添加一个type，如下图。记得在这里的URL Schemes的内容你可以填入任意值，当时需要在下面地方保持一致。否则会导致APP跳入到支付宝而无法返回应用。
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDKURLType.png)
 * 在新工程的info.plist中添加如下。注意 URL Schemes 要与刚才的URL Types的URL Schemes的保持一致。如下图
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDKInfo.png)
 * 添加相应的库
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDKLib.png)
 * 添加预编译文件.pch并指明pch文件的相对路径。文件包含了一下内容即可。关于pch自行了解
 * 修改工程中的NSString *appScheme = @"alipayPayDemo"。也就是跟前面的保持一致的名字，一共有三个地方
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySchemes.png)
 * build一下，编译通过！！
 * 这时候，仿照SDK demo的APViewcontroller.m中添加一个支付方法，填入自己的三个key再run并完成支付。
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipayKey.png)
 
 #### 整体项目结构
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/AlipaySDKArchi.png)
 
 ####总结
 集成过程中，由于一直没有使用pch文件包含基本的 <UIKit/UIKit.h>，<Foundation/Foundation.h>导致编译出错。其他都是一次性完成。





##2015-11-20

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/GPUImage.jpg)


> have fun with coreImage and GPUImage.

## GPUImage

###添加GPUImage

 * git clone [GPUImage](https://github.com/BradLarson/GPUImage.git)
 * 将这个项目拷贝到工程的目录下
 	![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120gpuimage.png)
 * 打开项目的framework目录，将**GPUImage.xcodeproj**直接拉到工程中
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120project.png) 
 * 文档工程中，选择TARGETS--Build Phases--Target Dependencies--点击**+**加号添加**GPUImage**,**注意图片是白色的小房子**
 * 继续同一个页面下面，Link Binary With Libraries添加图中相对应的静态库
 
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120lib.png)
 * 指定**库的路径**。选择PROJECT-Build Setting--搜索:**Header Search Paths**,双击后面的路径。点击加号。填上**framework/Source**的**绝对路径**或者**相对路径**，最后点击选择**recursive**。使用**相对路径**，则填上**$(SRCROOT)/masonry/lib/GPUImage/framework/Source**,最后点击选择**recursive**.**$(SRCROOT)**就是你工程最顶层的目录。
 
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120path.png)
 * 最后，在工程的ViewController.m添加**#import "GPUImage.h"**。And run这个工程。
 
###测试GPUImage

 
	GPUImageSepiaFilter *filter = [[GPUImageSepiaFilter alloc] init];
    self.catImageView.image = [filter imageByFilteringImage:[UIImage imageNamed:@"cat.jpg"]]; 
    
###测试结果
	
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120result.png)

### 根据GPUImage的demo学习滤镜
	
####拍照图片添加滤镜
**SimplePhotoFilter**

####视频录像添加滤镜
**SimpleVideoFileFilter**，视频的文件可以使用Xcode直接读取沙盒的文件得到对应的录像文件。

#####更多的学习可以直接在对应的github文件中获得。
---

##2015-11-07

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/encourage.jpg) 

### coreText框架学习

> 学习coreText，让我想起了大学那段学习液晶屏幕驱动显示的时光。

 * 原文地址 [f* GFW](http://www.zoomfeng.com/blog/coretextshi-yong-jiao-cheng-%5B%3F%5D.html)
 
 
####概览

#####简介
 coteText是一个apple的文版排版框架。它直接将文本的内容，颜色，字体等全部属性交给coreGraphics进行渲染。由于直接与coreGraphics交互，因此渲染非常的高效。
 
#####组成架构
 
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/coreTextArchi.png) 

#####coteText与WebView在排版的差异

*  coretext占用内存更少，渲染更快。
*  coretext在渲染之前就已经知道了文本的全部属性，其中包含了文本的高度等等属性。但是WebView只可以在渲染之后才能知道如何文本的高度。
* coretext毫无疑问的是具有最好的原生交互效果。然后WebView只可以通过JS进行深度交互，而且交互效果效果也是比不上原生的。
* 但是coretext并不能想WebView那样支持文本的复制与粘贴。
* coretext在排版上面的逻辑更加复杂。当然，你想控制的地方更多更加深入，逻辑的复杂度也是伴随而来的。




####绘制纯文本

	 // 1.获取上下文
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    
    // 2.转换坐标系
    CGContextSetTextMatrix(contextRef, CGAffineTransformIdentity);
    CGContextTranslateCTM(contextRef, 0, self.bounds.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    
    // 3.创建绘制区域，可以对path进行个性化裁剪以改变显示区域
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, self.bounds);
    
    // 4.创建需要绘制的文字
    NSMutableAttributedString *attributed = [[NSMutableAttributedString alloc] initWithString:self.text];
    
    // 设置行距等样式
    [[self class] addGlobalAttributeWithContent:attributed font:self.font];
    
    // 加点料
    [attributed addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(10, 5)];
    
    [attributed addAttribute:NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(5, 10)];
    
    // 5.根据NSAttributedString生成CTFramesetterRef
    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributed);
    
    CTFrameRef ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attributed.length), path, NULL);
    
    // 6.绘制
    CTFrameDraw(ctFrame, contextRef);
    
   
#####1 获取上下文

	CGContextRef contextRef = UIGraphicsGetCurrentContext();
   就是获取当前画布的上下文，因为会将这个上下文传递给coreGraphics进行渲染的。
上下文，这个词第一次出现我大学学习RTOS的时候。当系统切换线程的时候，就保存当前线程的上下文到栈里面，然后又从栈中得到将要切换到的线程的上下文并进行执行。哈哈，当初翻译这个context的前辈是怎样想到这个词的呢?好奇怪但是又觉得好贴切。

#####2 切换坐标系
	CGContextSetTextMatrix(contextRef, CGAffineTransformIdentity);
因为coreGraphics的原点是在左下角，coretext的是在左上角。因此，要对画布用当前的矩阵进行翻转坐标系。在代码中我们可以log出每个CTRun的rect.origin.y就可以知道是从最底部开始draw到最顶部的。
	
	CGContextTranslateCTM(contextRef, 0, self.bounds.size.height);
将当前的画布的原点平移到**{0，self.bounds.size.height}**

如果没有进行翻转坐标系，会发现文本产生镜像式的倒转显示。

##### 3 创建绘制区域
	    CGMutablePathRef path = CGPathCreateMutable();
        CGPathAddRect(path, NULL, self.bounds);
        
对path进行个性化裁剪从而改变显示区域。如果代码的一样，一般都是与当前view的bounds保持一致，就是满显示。

##### 4 创建需要绘制的文本
	NSMutableAttributedString *attributed = [[NSMutableAttributedString alloc] initWithString:self.text];
    
 这时候，准备开始生产东西了，当然就是从文本这个材料开始了。
 
 	[[self class] addGlobalAttributeWithContent:attributed font:self.font];
 	
 设置文本的全局属性，比如行距，字体大小，默认颜色。当然后面可以对文本进行再次的加工
 
 	[attributed addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(10, 5)];
    
   	 [attributed addAttribute:NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(5, 10)];


#####5 根据前面的材料生产得到最终渲染的全部属性材料
 
	CTFramesetterRef framesetter =CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributed);
	
先用之前的AttributeString创建CTFramesetterRef

	CTFrameRef ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attributed.length), path, NULL);
    
再用上面创建的frameRef以及path作为参数创建得到最后的渲染的frame

#####6 绘制
	CTFrameDraw(ctFrame, contextRef);
    
#### 图文混排

---

从纯文本的排版到图文混排，需要知道的是整个文本的结构。

 * CTLine:每一行的文本
 * CTRun：每一行不同的属性的文本


如何实现图文混排呢？其实就是先用一个占位符填充到需要显示图片的位置，并设置这个空白占位符的宽高等。当图片下载或者从本地读取之后就使用平时渲染Image的方式渲染到对应的地方。
从代码中可以知道对于**本地**以及**网络**上的图片，设置占位符的方式是不一样的。**本地**或者**网络**的使用**" "**作为占位符，也可以使用**0xfffc**。最后都是通过遍历整个文本的CTRun来得到图片的位置，根据这个CTRun的属性使用**CGContextDrawImage**进行渲染。

####调整文本高度

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/word.png) 

	* 根据文本保持baseLined对齐，高度是根据每一行的CTRun的属性确定。在代码中，第一行是不需要计算文本对齐的，而是从第二行开始的。
	* 根据文本自定义高度，实现整个文本的统一高度
	
####实现文本的点击交互
	思路:为这个view添加手势事件，然后通过手势获取当前屏幕的**点击点的位置**。然后，在这个手势事件中使用正则表达式来获取整个文本中会出发点击事件的**文本块**，最后通过匹配**点击点的位置**与**文本块**，**点**是否落在某个**文本块**中，最后对具体的**块**做出相应的事件出发


##2015-10-08

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151008.jpg) 

###UIWebView与JS的深度交互


>  博主一直以来是我的学习榜样，很多很好的解决方法。

 * 原文地址[f*ck GFW](http://kittenyang.com/webview-javascript-bridge/)

记得以前刚开始的CRM系统也是使用JS来开发的。当时自己也是尝试解决原生与JS之间的交互问题，但是效果还不是很理想。思路其实也是跟当前博主的思路一样的， 可惜当时自己的能力与任务繁重问题导致没有更好地去解决。
 曾记得，之前微博也是转发了一个第三方库用来解决原生与JS之间的深度交互，需要找找。**最后也是暴露了自己需要找个时间来学习前端开发才可以了。**


=======
##2015-09-25

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150925.jpg) 

> iOS Block vs delegate

 * 原文地址: [f*ck GFW](http://blog.stablekernel.com/blocks-or-delegates/),在CocoaChina中也有对应的[译文](http://www.cocoachina.com/ios/20150925/13525.html)
 

###从delegate到Block

刚开始学习不同页面的时候,当然学习的是使用代理传值了.对于刚开始的学习者来说这种比较容易理解.后面学习使用AFNetworking,才看到别人家是使用Block来传值的.之后在自己的开发的项目中也使用过Block传值.用在对AFN的一些方法进行了进一步的封装以及两个界面之间的传值.

###什么时候使用Block,什么时候使用delegate呢?

跟着上面的描述,用了一段时间之后自己心中也提出了这样一个问题.虽然一直都会去搞清楚,似乎还是似懂非懂.也是可能是基础还是很差呢,在钻牛角尖!aha~今天特地去stackoverflow找到这么一段非常好的[回答](http://stackoverflow.com/questions/21771606/objective-c-delegate-or-c-style-block-callback).

>Each has its use.

Delegates should be used when there are ***multiple "events"*** to tell the delegate about and/or when the class needs to get data from the delegate. A good example is with **UITableView**.

A block is best used when there is ***only one (or maybe two) event***. A **completion block** (and maybe a failure block) are a good example of this. A good example is with NSURLConnection sendAsynchronousRequest:queue:completionHandler:.

A 3rd option is notifications. This is best used when there are possibly multiple (and unknown) interested parties in the event(s). The other two are only useful when there is one (and known) interested party.


从上面的回答中我们可以知道对于需要传递多个不同的数据时候使用代理代理,就像举例说的**UITableview**,一个代理包含可以声明多个代理方法实现不同数据的传递.对于只有单一事件或者数据传递可以直接使用Block,我自己也是认为在这种情况使用Block可以更加简洁.比如在网络请求部分就是使用了Block,非常简洁明了.

###那么对于多人开发的时候呢?应该选择代理还是Block?

这个问题我实在微博看到别人对这边[译文](http://www.cocoachina.com/ios/20150925/13525.html)的一个评论.暂时没有思考得到其中的原因.为什么这么提问,怎样去回答??先留着.

---


##2015-09-20

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150920.jpg) 


>UIScrollView 实践经验

 * 原文地址:这篇文章不需要爬梯  [f*ck GFW](http://tech.glowing.com/cn/practice-in-uiscrollview/)
 
如标题所讲,是关于UIScrollView的实践经验的.读罢全篇文章,我觉得这是对UISCrollview的最佳实践之一了.当我看到这篇文章的时候,一个词:**awesome**
 
文中是从UIScrollView的关于触摸,滑动等事件以及对应过程的代理协议出发,讨论了**UITableView**(UIScrollView的子类)在图片优化方面的处理.UIScrollView的**分页实现方式**,竟然还有**ViewController重用**等问题.其中关于**UITableView**的优化让我收获最多.同时,我自己也跟着里面的分析过程写了一个测试[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git).

在[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git)中,我并没有使用文章中使用SDWebImage来请求图片并测试.而是使用最简单的检测方法.详情请看[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git)的实现过程.当然你也可以直接到一些图片网站测试更加好了,比如我之前使用的是[500px](http://www.jianshu.com/p/f1208b5e42d9) , 这个就是当时学习**swift**的网络请求库**alamoFire**找到的.当然,这个关于**alamoFire**的两篇教程质量还是杠杠的.So,enjoy it!
	
	cell.textLabel.text = [NSString stringWithFormat:@"row:%d",indexPath.row];
	
当然最好是按照文中提到的**问题1,2,3**一步步来跟着分析过程来测试.记得好好留意这个代理方法:

    - (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset
    
最后多谢博主的分享.happy everyday!!

---
=======

##2015-09-12

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150912.jpg)   

>Swift:The rules of being weak and unowned in clusures and capture lists(Xcode 6 Beta 7)

### swift：weak和unowned关键字在闭包以及捕获列表中的使用规则
* 原文地址(需要自备梯子):[f*ck GFW](http://sketchytech.blogspot.com/2014/09/swift-rules-of-weak-and-unowned.html)

从早前文章提出的**weak**,**wnowned** 和 **lazy**关键字，我应该在在这里好好地思考它们在**闭包**中的使用方法

####一、为什么我们要考虑如何在闭包中使用weak以及unwonted？

第一件需要做的事情就是解释为什么我们要再**闭包**中使用weak和**unowned**.然而那些牛人已经对此写好了文档，因此我们只要阅读这些文档就可以知道上述问题的所在了

>如果你要用一个闭包为一个类的实例中的属性赋值，此时闭包就会通过引用这个实例或者或者该实例的成员来捕获得到这个实例。那么你就会在闭包与实例之间建立了强引用循环。Swift语言就使用捕获列表来打破这种强引用循环 ---		Apple

####二、如何利用捕获列表来解决问题呢？
这令人感觉到一种强烈的责任感而不去在一个闭包中建立强引用循环。但是，只要我们花费一些些时间去理解这个捕获列表就可大大缓解这种焦虑感：

>一个闭包表达式可以指定通过捕获列表来捕获周围得到的值。捕获列表写在一个方括号里，不同的值之间使用逗号分隔开。如果要你需要用到捕获列表，就必须使用**in**关键字，即使你漏掉了参数的名称，参数的类型以及返回值的类型。

每一个捕获列表的入口，都使用**marked**或者**unowned**来标记使用它们引用的值

	myFunction { print(self.title) }                    // strong capture
	myFunction { [weak self] in print(self!.title) }    // weak capture
	myFunction { [unowned self] in print(self.title) }  // unowned capture

在捕获列表中，你也可以使用一个随机表达式来命名一个值。当一个闭包形成时就会计算这些表达式以及得到它指定的强弱引用程度，比如：

	// Weak capture of "self.parent" as "parent"
	myFunction { [weak parent = self.parent] in print(parent!.title) }" (Apple)


#### 三、在闭包中weak和unowned的使用规则
首先，最重要的是要弄清楚只有在相关的闭包中哪个地方我们要用一个闭包为类的实例赋值。因此，我们要记得一下这些规则

* 1 如果类的实例或者属性是可选类型的，就使用**weak**
* 如果类的实例或者属性不是可选类型并且永远都不会被赋值为nil,使用**unwonted**
* 必须使用**in**关键字，即使你没有参数名称，参数类型或者返回值类型
*

**Note:** 在下面的列子中的闭包都是关于**lazy**属性变量的。因为在其他的情况下它将是不可访问的属性变量。除非这个类已经被初始化了。

##### 四、解释和测试代码
一个强引用的闭包：

	class Parent {
    	var title = "Dad"
    	lazy var parentOf:(String)->String = {
    		(childname:String) in return childname+"'s "+self.title
    	} // DON'T DO THIS UNLESS YOU WANT THE CLOSURE TO KEEP THE INSTANCE ALIVE!!!
	 }
	 
因此，我们可以添加一个捕获列表来解决这个问题：

	class Parent {
    	var title = "Dad"
    	lazy var parentOf:(String)->String = {
   	 [unowned self] (childname:String) in return childname+"'s "+self.title
    	} // OK
 	}
 	
**Note:** 这个捕获列表是属于**[unowned self]**部分

#### 五、 unowned和weak的区别

即使**title**变量是可选类型的，但是我们仍然可以使用**owned**

	class Parent {
   	 var title:String? = "Dad"
   	 lazy var parentOf:(String)->String = {
   	 	[unowned self](childname:String) in return childname+"'s "+self.title!
    		}
	}
	
因为**title**是一个可选类型的字符串变量，而不是一个可选类型的类实例。然而，让我们假设一下创建一个**child**类并包含一个可选类型的**parent**属性变量：

	class Child {
    var parent:Parent?
    init (parent:Parent) {
        self.parent = parent
    }
    lazy var myParent:()->() = { [weak parent = self.parent] in print(parent!.title) }
	}
 
现在，我们这里使用的是**weak**，因为**parent**是一个可选类型的类实例变量。我们同样使用了这个语法来引用了**self**并允许去指定属性。需要记住的是在闭包中如何使用**parent**而不是**self.parent**.(Tip:尝试将**weak**用**unowned**替换，编译器会提示错误的)

#### 六、同时使用**weak**和**unowned**
让我们继续添加一个不是可选类型的**grandparent**变量。我们可以简单地添加它到捕获列表中（此时，我们使用的是**unowned**因为它是一个非可选类型的变量）

	class Child {
    var parent:Parent?
    var grandparent:GrandParent
    init (parent:Parent, grandparent:GrandParent) {
        self.parent = parent
        self.grandparent = grandparent
    }
    lazy var myParent:()->() = { [weak parent = self.parent, unowned grandparent = self.grandparent] in print("\(parent!.title) is \(grandparent.title)'s son"); }
	}
	
另外，这个是grandpaent的类的定义

	class GrandParent {
    var title:String = "Gramps"
	}
	
接下来就是一些基本的使用方式

	let kid = Child(parent: Parent(), grandparent:GrandParent())
	kid.myParent()
	
#### 总结
