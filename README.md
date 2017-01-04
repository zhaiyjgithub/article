# article
some article about program.
***

##20161226
本章将学习**灯光**
**从本章开始，对应章程的代码将不会重新上传，可以直接下载 http://www.cosmicthump.com/learning-opengl-es-sample-code 全部的源码并对应章节查阅即可**

在本章节中，对应的是源码**OpenGLES_Ch5_2**的工程。你会发现，在工程中，不会像以前那样自定义几个三角形的顶点。取而代之的是一个**球体模型**，一个**球体模型**可以由**多个三角形组成的网格组成的**。那么只需要得到组成的球体模型的所有网格三角形的顶点即可。然而，这个模型就是需要工具**Blender**来生成的。关于**模型的生成**将在后面的章节会讲解到。当前先使用已经弄好的网格模型，暂不深究模型的制作。

###代码分析

**灯光处理**
    
      self.baseEffect.light0.enabled = GL_TRUE;
      self.baseEffect.light0.diffuseColor = GLKVector4Make(
         0.7f, // Red 
         0.7f, // Green 
         0.7f, // Blue 
         1.0f);// Alpha 
      self.baseEffect.light0.position = GLKVector4Make(
         1.0f,  
         0.0f,  
        -0.8f,  
         0.0f);
      self.baseEffect.light0.ambientColor = GLKVector4Make(
         0.2f, // Red 
         0.2f, // Green 
         0.2f, // Blue 
         1.0f);// Alpha 

**ambientColor：**设置环境光的颜色
**diffuseColor：**设置漫反射光的颜色
**position：**设置灯光的位置。这时候请联想起OpenGL ES的坐标模型就知道灯光点在哪个位置发射光线出来。同样你可以更改参数来体会光线位置就更佳直观了。

代码的其他地方也是跟以前的工程一样，申请缓存并写入顶点数据。

###总结
灯光和纹素一样，可以包含两个纹素和灯光。

##20161226
本章将学习**多重纹理**

很多有用的可视效果可以通过片元颜色（就是纹理）和在像素颜色渲染颜色缓存中现存的颜色相混合来实现。但是这个技术有两个主要的缺点：每次显示更新时，几何图形必须要被渲染一次或者多次，混合函数需要从像素颜色缓存中读取颜色数据以便与片元颜色混合，然后结果被写回帧混存。当带有透明度数据和多个纹理层叠时候，每个纹理的像素颜色渲染缓存的颜色就会被再次读取、混合、重写。其实这个效果就是**多通道渲染**。
    当前CPU都是支持同时从至少两个纹理缓存中采样纹素。
    
####代码分析
先回到viewDidLoad方法中的后面，增加一个新的纹理。

    CGImageRef imageRef0 =
    [[UIImage imageNamed:@"leaves.gif"] CGImage];
    
    GLKTextureInfo *textureInfo0 = [GLKTextureLoader
                                    textureWithCGImage:imageRef0
                                    options:[NSDictionary dictionaryWithObjectsAndKeys:
                                             [NSNumber numberWithBool:YES],
                                             GLKTextureLoaderOriginBottomLeft, nil]
                                    error:NULL];
    
    self.baseEffect.texture2d0.name = textureInfo0.name;
    self.baseEffect.texture2d0.target = textureInfo0.target;
    
    //添加纹理
    CGImageRef imageRef1 =
    [[UIImage imageNamed:@"beetle.png"] CGImage];
    
    GLKTextureInfo *textureInfo1 = [GLKTextureLoader
                                    textureWithCGImage:imageRef1
                                    options:[NSDictionary dictionaryWithObjectsAndKeys:
                                             [NSNumber numberWithBool:YES],
                                             GLKTextureLoaderOriginBottomLeft, nil] 
                                    error:NULL];
    
    self.baseEffect.texture2d1.name = textureInfo1.name;
    self.baseEffect.texture2d1.target = textureInfo1.target;
    self.baseEffect.texture2d1.envMode = GLKTextureEnvModeDecal;

**self.baseEffect.texture2d1.envMode = GLKTextureEnvModeDecal;**这里配置纹理的混合模式

最后在准备渲染纹理坐标的时候，添加第二个纹理的坐标即可。

    
    - (void)glkView:(GLKView *)view drawInRect:(CGRect)rect
    {
        // Clear back frame buffer (erase previous drawing)
        [(AGLKContext *)view.context clear:GL_COLOR_BUFFER_BIT];
        
        [self.vertexBuffer prepareToDrawWithAttrib:GLKVertexAttribPosition
                               numberOfCoordinates:3
                                      attribOffset:offsetof(SceneVertex, positionCoords)
                                      shouldEnable:YES];
        
        [self.vertexBuffer prepareToDrawWithAttrib:GLKVertexAttribTexCoord0
                               numberOfCoordinates:2
                                      attribOffset:offsetof(SceneVertex, textureCoords)
                                      shouldEnable:YES];
        
        [self.vertexBuffer prepareToDrawWithAttrib:GLKVertexAttribTexCoord1
                               numberOfCoordinates:2
                                      attribOffset:offsetof(SceneVertex, textureCoords)
                                      shouldEnable:YES];
        
        [self.baseEffect prepareToDraw];
        
        // Draw triangles using currently bound vertex buffer
        [self.vertexBuffer drawArrayWithMode:GL_TRIANGLES
                            startVertexIndex:0
                            numberOfVertices:sizeof(vertices) / sizeof(SceneVertex)];
    }

###总结
上面就是多重纹理使用，主要是要清楚使用多重纹理在一个通道的优点即可。

##20161216
本章将学习**采样方式对纹理的影响**。

经常会出现这样的一种情况：加载了一个拥有很多纹素的纹理缓存，去渲染一个只是拥有几个像素的三角形的情况。也就是一个小盒子要放很多小物件的情况。同样地，也会出现相反的情况。

在前面的情况下，我们应该使用怎样的计算方法，从同样映射到同一个像素的一堆纹素中选择得到最好效果的纹素呢？这时候就是涉及到**采样算法**。

采样算法包含了两种：**Linear(线性采样)、neatest(最近采样)**。

**Linear(线性采样)：**

*  **纹素多，像素少：**当多个纹素对应一个像素时，OpenGL ES就会自动从这对匹配的纹素中，使用**线性内插法**混合这些颜色然后得到最后的颜色。比如两个纹素黑色和白色，那么线性采样之后就时灰色。
*  **纹素少，像素多：**OpenGL ES会缓和附近的像素得到最好的颜色，并放大该纹理。最后的效果就是**模糊渲染到三角形**。

**neatest(最近采样)：**

* **纹素多，像素少：**用最靠近该像素点的纹素作为最后的颜色被采样。
* **纹素少，像素多：**OpenGL ES仅仅只会获取距离该像素点最近的一个纹素，然后放大该纹理。最后的感觉就是**像素化渲染到三角形**。

###代码分析

##20161209
本章开始学习一下**纹理**。之前渲染的三角形都是只有一种颜色，如果我们需要在三角形上面添加图片呢？

概念：**纹理是一个用来保存图像的颜色元素值的OpenGL ES缓存。**当用一个图像初始化一个纹理缓存之后，在这个图像中的每个像素变成纹理中的一个**纹素**。纹素存在于一个没有单位的数学坐标系中，而且是只有正值**{S,T}坐标系。**。

那么，前面我们指定的三角形坐标，如何从纹素中获取对应的纹素呢？这时候就需要用到**坐标映射**。只需要在定义三角形的顶点坐标的时候，同时指定纹素的映射坐标即可，在顶点指定的映射坐标系叫做**{U,V}坐标系。**

上面的**{S,T}坐标系，{U,V}坐标系都是从左下角作为原点，上方为T，V轴，右方S，U轴。**

###代码分析
首先，我们从上面一个工程的基础上进行代码修改。
####拓展顶点
在前面已经讲到，需要在顶点坐标的定义，同时制定纹素的坐标。

    typedef struct{
        GLKVector3  positionCoords; //顶点坐标{x,y,z}
        GLKVector2  textureCoords;//纹理映射坐标{u,v}
    }
    SceneVertex;

首先重新定义顶点坐标结构体。

    //定义包含映射到纹理的坐标
    static const SceneVertex vertices[] ={
        {{-0.5f, -0.5f, 0.0f}, {0.0f, 0.0f}}, // lower left corner
        {{ 0.5f, -0.5f, 0.0f}, {1.0f, 0.0f}}, // lower right corner
        {{-0.5f,  0.5f, 0.0f}, {0.0f, 1.0f}}, // upper left corner
    };
    
从上面拓展的坐标中，可以知道三个纹素坐标也构成了一个**三角形**。那么，每个纹素坐标对应于三角形每个顶点坐标。然后，OpenGL ES就会从纹理缓存中读取纹素渲染到三角形中对应的坐标位置中去。其实，**顶点的三角形和纹素坐标成的三角形是一个相似的三角形来的。**你会问，如果纹素组成的三角形的面积比顶点组成的三角形的面积少呢？也就是，纹素数量少于需要渲染的空间？这时候，就涉及了**采样算法了**。这个先留在下一章讲解。

####生成纹理

        //使用一章图片生成一个2D纹理
    CGImageRef imageRef = [[UIImage imageNamed:@"leaves.gif"] CGImage];
    GLKTextureInfo * textureInfo = [GLKTextureLoader textureWithCGImage:imageRef options:nil error:NULL];
    //将纹理赋给baseEffect
    self.baseEffect.texture2d0.name = textureInfo.name;
    self.baseEffect.texture2d0.target = textureInfo.target;
    
回到viewDidLoad方法中，在最后添加上面的代码。首先，使用**GLKTextureLoader**将一张图片生成一个纹理对象**GLKTextureInfo。**然后将纹理赋值给baseEffect。

####准备纹理坐标

      //准备好纹理坐标
    [self.vertexBuffer prepareToDrawWithAttrib:GLKVertexAttribTexCoord0 numberOfCoordinates:2 attribOffset:offsetof(SceneVertex, textureCoords) shouldEnable:YES];
    

然后来到**- (void)glkView:(GLKView *)view drawInRect:(CGRect)rect**，在绘制之前添加上面的代码，上面的代码就是准备好纹理坐标。接下来，进行绘制三角形，然后OpenGL ES从纹素缓存中读取缓存并渲染到对应的位置中。

#####总结
拓展顶点的坐标，映射到纹理坐标中，获取对应位置的纹素渲染到顶点三角形中。最后得到一个有纹理的坐标。
**尝试，同时更改顶点的{U,V}坐标，保持两个三角形相似性，看看最后的渲染的效果会怎样？最后，你会发现，顶点三角形就相当于一个窗口，移动{U,V}坐标就相当于窗口的位置，看到图片其他部分的颜色罢了。**


##20161207
本次的文章主要是封装上一章的代码，比如如缓存的申请、绑定和数据保存。这也是因为后面的教程都会在当前这份代码的基础上进行开发的。

###上下文context的方法相关封装
新建两个文件： **AGLKContext.h** 和**AGLKContext.m**
头文件的内容：

    #import <GLKit/GLKit.h>

    @interface AGLKContext : EAGLContext

    @property(nonatomic,assign)GLKVector4 clearColor;
    - (void)clear:(GLenum)mask;
    - (void)enable:(GLenum)capability;
    - (void)disable:(GLenum)capability;

    @end
    
.m文件的内容：

    #import "AGLKContext.h"

    @implementation AGLKContext
    
    
    - (void)setClearColor:(GLKVector4)clearColor{
        _clearColor = clearColor;
        
        glClearColor(clearColor.r, clearColor.g, clearColor.b, clearColor.a);
    }
    
    - (void)clear:(GLenum)mask{
        glClear(mask);
    }
    
    - (void)enable:(GLenum)capability{
        glEnable(capability);
    }
    
    - (void)disable:(GLenum)capability{
        glDisable(capability);
    }
    @end
    
上面主要封装了context一些关于设置当前上下文背景颜色相关的代码。

###封装缓存的申请和绑定相关
新建两个文件： **AGLKVertexAttribArrayBuffer.h** 和 **AGLKVertexAttribArrayBuffer.m**

头文件的内容：
    
    #import <Foundation/Foundation.h>
    #import <GLKit/GLKit.h>
    
       typedef enum{
           AGLKVertexAttribPosition = GLKVertexAttribPosition,
           AGLKVertexAttribNormal = GLKVertexAttribNormal,
           AGLKVertexAttribColor = GLKVertexAttribColor,
           AGLKVertexAttribTexCoord0 = GLKVertexAttribTexCoord0,
           AGLKVertexAttribTexCoord1 = GLKVertexAttribTexCoord1,
       } AGLKVertexAttrib;
       
       @interface AGLKVertexAttribArrayBuffer : NSObject
       @property(nonatomic,assign)GLsizei stride; //步幅
       @property(nonatomic,assign)GLsizeiptr bufferSizeBytes; //顶点等缓存数据指针
       @property(nonatomic,assign)GLuint name;//缓存ID
       
       - (id)initWithAttribStride:(GLsizei)aStride numberOfVertices:(GLsizei)count bytes:(const GLvoid *)dataPtr usage:(GLenum)usage;
       - (void)reinitWithAttribStride:(GLsizei)aStride numberOfVertices:(GLint)count bytes:(const GLvoid *)dataPtr;
       - (void)prepareToDrawWithAttrib:(GLint)index numberOfCoordinates:(GLint)numberOfCoordinates attribOffset:(GLsizeiptr)offset shouldEnable:(BOOL)shouleEnable;
       - (void)drawArrayWithMode:(GLenum)mode startVertexIndex:(GLint)first numberOfVertices:(GLint)count;
       @end
    
    上面的代码枚举了原来内置的GLKVertexAttrib作为区分，并定义**stride**、**bufferSizeBytes**、**name**等相关变量。
    
    .m文件的内容：
    
        #import "AGLKVertexAttribArrayBuffer.h"
    
    @implementation AGLKVertexAttribArrayBuffer
    
    
    /**
      把顶点数据保存到申请的缓存中
    
     @param aStride 步幅
     @param count 顶点的个数
     @param dataPtr 顶点坐标的数组
     @param usage <#usage description#>
     @return <#return value description#>
     */
    - (id)initWithAttribStride:(GLsizei)aStride numberOfVertices:(GLsizei)count bytes:(const GLvoid *)dataPtr usage:(GLenum)usage{
    
        if (self = [super init]) {
            self.stride = aStride;
            self.bufferSizeBytes = count * self.stride;
            
            glGenBuffers(1, &_name);
            glBindBuffer(GL_ARRAY_BUFFER, self.name);
            glBufferData(GL_ARRAY_BUFFER, self.bufferSizeBytes, dataPtr, usage);
            
            NSAssert(0 != self.name, @"failed to generate buffer name"); 
        }
        return self;
    }
    
    
    /**
     重置当前绑定的缓存数据
    
     @param aStride <#aStride description#>
     @param count <#count description#>
     @param dataPtr <#dataPtr description#>
     */
    - (void)reinitWithAttribStride:(GLsizei)aStride numberOfVertices:(GLint)count bytes:(const GLvoid *)dataPtr{
        self.stride = aStride;
        self.bufferSizeBytes = count * self.stride;
        
        glBindBuffer(GL_ARRAY_BUFFER, self.name);
        glBufferData(GL_ARRAY_BUFFER, self.bufferSizeBytes, dataPtr, GL_DYNAMIC_DRAW);
    }
    
    
    /**
     使能坐标读取以及指定顶点坐标的组成等参数
    
     @param index <#index description#>
     @param numberOfCoordinates <#numberOfCoordinates description#>
     @param offset <#offset description#>
     @param shouleEnable <#shouleEnable description#>
     */
    - (void)prepareToDrawWithAttrib:(GLint)index numberOfCoordinates:(GLint)numberOfCoordinates attribOffset:(GLsizeiptr)offset shouldEnable:(BOOL)shouleEnable{
        glBindBuffer(GL_ARRAY_BUFFER, self.name);
        
        if (shouleEnable) {
            glEnableVertexAttribArray(index);
        }
        
        glVertexAttribPointer(index, numberOfCoordinates, GL_FLOAT, GL_FALSE, (self.stride), NULL + offset);
    }
    
    
    /**
     绘制模型
    
     @param mode <#mode description#>
     @param first <#first description#>
     @param count <#count description#>
     */
    - (void)drawArrayWithMode:(GLenum)mode startVertexIndex:(GLint)first numberOfVertices:(GLint)count{
        glDrawArrays(mode, first, count);
    }
    
    @end

**- (id)initWithAttribStride:(GLsizei)aStride numberOfVertices:(GLsizei)count bytes:(const GLvoid *)dataPtr usage:(GLenum)usage**：这个方法就是初始化步幅，顶点的数量，顶点数据的指针以及数据的使用方式。看改方法的内容可以知道里面就是使用传递进来的参数申请缓存ID和把顶点数据保存到缓存中。

**- (void)reinitWithAttribStride:(GLsizei)aStride numberOfVertices:(GLint)count bytes:(const GLvoid *)dataPtr**：重置缓存中的数据。这是为了以后使用，当前工程中并没有使用改方法。

**- (void)prepareToDrawWithAttrib:(GLint)index numberOfCoordinates:(GLint)numberOfCoordinates attribOffset:(GLsizeiptr)offset shouldEnable:(BOOL)shouleEnable**：准备绘制模型数据之前的准备工作。里面的内容就是上一个工程的代码，在绘制代理方法中关于绘制前准备代码的封装

**- (void)drawArrayWithMode:(GLenum)mode startVertexIndex:(GLint)first numberOfVertices:(GLint)count**：绘制模型。指定绘制的起始点和顶点的数量。

###引入并调用
回到**viewDidLoad**方法中，调用的方式如下：
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        
        GLKView * view = (GLKView *)self.view;
        NSAssert([view isKindOfClass:[GLKView class]], @"view controller`s view is not glkView");
        
        view.context = [[AGLKContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
        [AGLKContext setCurrentContext:view.context];
        
        self.baseEffect = [[GLKBaseEffect alloc] init];
        self.baseEffect.useConstantColor = GL_TRUE;
        self.baseEffect.constantColor = GLKVector4Make(1.0, 1.0, 1.0, 1.0);
        
        ((AGLKContext *)view.context).clearColor = GLKVector4Make(0.0, 0.0, 0.0, 1.0);
        
        self.vertexBuffer = [[AGLKVertexAttribArrayBuffer alloc]
                             initWithAttribStride:sizeof(SceneVertex)
                             numberOfVertices:sizeof(vertices)/sizeof(SceneVertex)
                             bytes:vertices usage:GL_STATIC_DRAW];
    
    }

对比上次的工程代码，你会发现只是修改了**context.clearColor**和**self.vertexBuffer的定义**。

再看看绘制的代理方法：
    
    - (void)glkView:(GLKView *)view drawInRect:(CGRect)rect{
    [self.baseEffect prepareToDraw];
    
    [(AGLKContext *)view.context clear:GL_COLOR_BUFFER_BIT];
    
    [self.vertexBuffer prepareToDrawWithAttrib:GLKVertexAttribPosition numberOfCoordinates:3 attribOffset:offsetof(SceneVertex, positionCoors) shouldEnable:YES];
    
    [self.vertexBuffer drawArrayWithMode:GL_TRIANGLES startVertexIndex:0 numberOfVertices:3];
    }
    
同样对比上次的工程代码。你会发现不同的是**步幅=attribOffset**使用了**offsetof(SceneVertex, positionCoors)**来获取**positionCoors在SceneVertex结构体中的偏移量**.在当前定义的结构体中我们可以知道**SceneVertex只有一个成员，那么偏移量就是0**。为什么使用这样的方式呢？**因为有些顶点的信息不单包含了顶点的三个空间坐标{x,y,z}，还可以包含其他的信息，比如下一章说道的纹理坐标{U,V}。如果包含了其他的坐标信息，那么当获取其他的坐标的信息时，传入的偏移量就不一样的了**。

###总结
本章内容主要是封装了**context**、**buffer**等相关代码。

##20161204

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20161204.jpg) 

> Beauty provoketh thieves sooner than gold

大半年没有写blog了，这半年也是太忙了。好吧！直接入正题，接下来我将会逐步更新关于**OpenGL ES**的入门教程。教程我是看了**OpenGL ES应用编程实践 iOS卷**的一遍之后，再次写的笔记。我认为经过多次阅读与实践，会等到更多的体会。内容相比书里面的内容会快而且精简一些，如果涉及一些概念，可以先阅读该书之后或者直接参考该书对应的章节。
    今天是第一篇**绘制三角形**。

###渲染、坐标系、缓存
**渲染**：用3D数据生成一个2D图像的过程就是渲染。

**坐标系**：在OpenGL ES中，坐标系是一个空间坐标系。它是符合右手定则。屏幕的中心点就是**原点**，屏幕中心点指向自己的方向就是**Y轴正方向**。屏幕的中心点并平行屏幕的**向右方向**就是**X轴正方向**。屏幕的中心点执行听筒并垂直**X轴**方向就是**Y轴正方向**。OpenGL ES坐标系的体验，可以在完成教程运行demo后，通过更新demo的坐标数组的数据并运行即可。另外，**OpenGL ES坐标系是没有单位表示的**。

**缓存**：GPU能够控制和管理的连续RAM。GPU在处理缓存中的数据的同时，CPU的程序仍然是可以继续运行的。也就是，GPU和CPU是异步的。比如本教程中，三角形的顶点就是保存在缓存中，等到CPU发指令到GPU说要**渲染**的时候，GPU就会从缓存中读取数据并进行渲染。

###准备工作

先创建一个工程，打开**viewController.h**，加入**GLKit**的头文件，并将当前的VC继承自**GLKViewController**：
  
    #import <UIKit/UIKit.h>
    #import <GLKit/GLKit.h>

    @interface ViewController : GLKViewController
    @end
    
然后打开**storyboard**,更改当前的**view**继承于**GLKView**。

###代码分析
打开viewcontroller.m，定义两个变量：

    @interface ViewController ()
    @property(nonatomic,assign)GLuint vertexBufferID;
    @property(nonatomic,strong)GLKBaseEffect * baseEffect;
    @end
    
**vertexBufferID**：在缓存中唯一的ID
**vertexBufferID**：OpenGL ES提供最基本的shader effect

    typedef struct {
    GLKVector3 positionCoords; //使用OpenGL ES 内置的数据类型，这个是一个联合体union
    }
    SceneVertex;

使用OpenGL ES 内置的数据类型来描述当前的**一个顶点**。这里为什么使用内置的数据类型呢？因为这样更加方便，可以直接进去看看**GLKVector3**的类型定义。里面是一个包含四个成员的结构体，而且刚好每个成员的包含了三个成员，比如第一个成员包含了**{ float x, y, z; }**三个成员。这三个成员刚好用来描述顶点坐标坐标在空间坐标系中的表示，更加方便我们的访问。

    static const SceneVertex vertices[] = {
    {{-0.5f, -0.5f, 0.0}}, // low er left corner
    {{ 0.8f, -0.5f, 0.0}}, // lower right corner
    {{-0.5f,  0.5f, 0.0}}  // upper left corner
    };
    
根据上面的数据类型来定义一个数组，里面包含了三个顶点，刚好组成了一个**三角形**。你可以分析一下里面的顶点坐标，结合刚才OpenGL ES的空间坐标系，就知道三角形大概是怎样表示的了。

     GLKView *view = (GLKView *)self.view;
    NSAssert([view isKindOfClass:[GLKView class]],
             @"View controller's view is not a GLKView");

断言当前的view是不是**GLKView 类型**。

     
    view.context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    
    [EAGLContext setCurrentContext:view.context];

创建一个OpenGL ES2.0的上下文到当前的view。然后将view的上下文设置到当前的上下文。

     
    self.baseEffect = [[GLKBaseEffect alloc] init];
    self.baseEffect.useConstantColor = GL_TRUE;
    self.baseEffect.constantColor = GLKVector4Make(1.0, 1.0, 1.0, 1.0);

创建一个基本的OpenGL ES 2.0 的effect 对象，并设置着色器使用基本的渲染颜色。

     //设置当前context的背景颜色
    glClearColor(0, 0, 0, 0);
    
    //获取一个独一无二的缓存ID
    glGenBuffers(1, &(_vertexBufferID));
    
    //将这个buffer ID绑定到一个申请的buffer中
    glBindBuffer(GL_ARRAY_BUFFER, _vertexBufferID);
    
    // 将顶点的数据保存在刚才申请的buffer中,因为当前的顶点数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
描述如注释。就是把上面定义的顶点数组数据保存在唯一标记的缓存中，以供后面GPU从缓存中读取数据并进行渲染到屏幕上。

    //屏幕每帧都会自动调用这个方法，同update，view的渲染使用glkView
    - (void)glkView:(GLKView *)view drawInRect:(CGRect)rect{
    
        [self.baseEffect prepareToDraw];
    
        //清除buffer之前的数据
        glClear(GL_COLOR_BUFFER_BIT);
    
        //使能从buffer读取顶点数据
        glEnableVertexAttribArray(GLKVertexAttribPosition);
    
        //设置顶点的数据指针 sizeof(SceneVertex) = 12 个字节 
        glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE,   sizeof(SceneVertex), NULL);
    
        //开始绘图
        glDrawArrays(GL_TRIANGLES, 0, 3);
    }
    
我们知道iPhone屏幕刷新的帧率60hZ，因为这是人眼识别的最好效果。超过了，就浪费资源，少了就会卡了。那么，GLKit刚好提供了一个跟帧率同步的代理方法：

    //屏幕每帧都会自动调用这个方法，同update，view的渲染使用glkView
    - (void)glkView:(GLKView *)view drawInRect:(CGRect)rect{
    
        [self.baseEffect prepareToDraw];
    
        //清除buffer之前的数据
        glClear(GL_COLOR_BUFFER_BIT);
    
        //使能从buffer读取顶点数据
        glEnableVertexAttribArray(GLKVertexAttribPosition);
    
        //设置顶点的数据指针 sizeof(SceneVertex) = 12 个字节 
        glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, sizeof(SceneVertex), NULL);
    
        //开始绘图
        glDrawArrays(GL_TRIANGLES, 0, 3);
    }


**glEnableVertexAttribArray** ：里面的参数表示顶点。可以进去看这个枚举定义OpenGL ES已经帮我们已经定义好了描述顶点，向量，颜色，纹理等。我们直接使用内置的枚举成员即可。
**glVertexAttribPointer**：第一个参数同上；第二个参数表示一个顶点包含了三个部分，就是**XYZ三个坐标点**；第三个参数表示坐标点的类型是浮点型的；第四个参数表示数组的顶点数据是否可以被改变的，本教程是不可改变的，那么就是false；第五个参数表示**步幅**，告诉GPU每次从缓存中读取一次数据获取多少个字节来表示一个完整的顶点数。而这里就是12个字节=步幅。为什么？因为一个float类型用4个字节来表示，64位系统为了兼容也是4个字节；最后一个参数表示数据读取从当前绑定的缓存的起点位置开始读取。
**glDrawArrays**：开始绘图。第一个参数告诉GPU描绘三角形；第二个参数表示三角形第一顶点在缓存中的位置；第三个顶点表示模型有三个顶点；

至此，我们先run一下工程，就会发现一个白色的三角形描绘在屏幕中间处（通过更改坐标系的坐标数据继续分析和体验OpenGL ES坐标系）。
    

###总结
流程就是定义**三角形模型**数据，把数据保存到申请的缓存中。然后在代理方法中，执行读取缓存数据并把数据渲染到屏幕上。
    

##20160425

##React与webpack入门

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/webpack.jpg) 

>No matter how long night, the arrival of daylight Association.

照旧，先上[源码](https://github.com/zhaiyjgithub/React-webpack-tutorial.git)

是的，我正在学习开发前端呢，时间将近一个多月了，但是都是晚上的时间(**白天都在写iOS，偶尔关注前端开发**)。为什么我会开始学习前端呢？首先，记得15年刚开始搞iOS开发，当时在公司一个OA项目有部分
功能是使用**webview**实现的。当时也是搞了一个礼拜，但是还是没有得到想要的效果。最后
我立马使用使用原生花费2个礼拜的时间全转了原生了。这个经历一直留在我的心中但是经过最近
一段时间才认真解决了原生与web交互的问题，也就是上一篇文章。同样地，在这个时间我学习了
HTML，CSS以及基本的JS，突然间发现前端也是很好玩的样子喔。。。。就开始制定学习计划(其实就是到知乎上面看大牛分享学习路线之后然后总结得到自己的学习路线)继续搞前面提到的知识，然后到jQuery学习jQuery和jQuery UI的。jQuery UI只是学习了一小部分吧，主要感觉它有点丑了。但是jQuery比较
喜欢，就是各种写小demo来测试学习吧，其中同时继续学习CSS基础知识。**接着，感觉越来越好了。。。**
	最近一个礼拜比较空闲，然后看了一下现在比较流行的三种框架:**Ng,Vue,React**。**Ng**很多人都是说它很重很大，作为刚入门的我先不打算它。**Vue**很轻量级，也在官网上面看了一下tutorial教程，简单容易上手，比较适合自己，重要的一点我很喜欢作者**尤老师**。再看**React**，很多人都说它的特色是**组件化**,同样地在官网的**getting start**这些tutorial来学习。在WS上面把玩一下，我马上发现**它就是我的菜了**。为什么，因为**组件化**就跟我平时**iOS开发自定义控件**方式很接近。接下来更具tutorial中的完整例子学习了**单向数据流**以及**组件间传值**等，对于其中**子组件传值到父组件使用注册的回调方法更加类比于我平时iOS开发的cell通过block传值到View Controller**，这些特点**一下子点爆了自己**。
	然后我在这个礼拜把官网的**getting start**的tutorial都玩一遍。我发现这样的组件化还不够好，必须继续学习一下工具来辅助开发才能更加提高效率。然后我就学习webpack，用它来**强化组件化**。
	好吧，不扯那么多，快快开始我们今天属于自己的 **Tutorial of React + webpack**吧!

### 安装npm
	
登录[node.js官网](https://nodejs.org/en/)下载安装包，我使用最新的stable version.
安装之后，打开terminal敲一下**npm**命令，试一下。

###安装全局webpack
在terminal敲: **npm install webpack -g**

###新建工程

* 建立一个工程(我使用的是WebStorm)，目录架构如下:

		|- app
		    |- main.js
		|- build
		    |- index.html
		|- components
		|- webpack.config.js
	
在终端cd到当前工程的根目录下,执行npm：**npm init**。之后就在根目录下生成一个package.json文件
这个文件包含了工程的基本信息，还会包含你的一些依赖信息，比如加载器有哪些。还可以添加执行的脚本，用于工程自动执行，后面就会说到。
目录变成了:

	|- app
	    |- main.js
	|- build
	    |- index.html
	|- components
	|- package.json
	|- webpack.config.js

* 配置webpack.config.js文件，在该文件中填写下面这些内容：

		var path = require('path');
		module.exports = {
			entry: path.resolve(__dirname, 'app/main.js'),
			output: {
	    	path: path.join(__dirname, 'build'),
	    	filename: 'bundle.js'
	    	},
		}
	
**entry** : 指定打包的入口文件
**output** : 指定打包后输出文件的路径以及文件

同样cd到根目录，执行: **webpack**。根目录下就会生成上面指定的输出文件:**build/bundle.js**。

添加相关加载器以及依赖包:**CSS**,**JSX**,**React**。分别执行命令:

**npm install webpack -save-dev**

**npm install jsx-loader css-loader style-loader --save-dev **
**npm install react react-dom --save-dev**.同样地，如果想要添加**jQuery**，替换 react即可。这些组件就会安装到当前工程的module文件夹下面。

* 重新配置一下**webpack.config.js**文件:

		var path = require('path');
		module.exports = {
			entry: path.resolve(__dirname, 'app/main.js'),
			output: {
				path: path.join(__dirname, 'build'),
				filename: 'bundle.js'     },
		    module: {
		    		loaders: [{
		    			test: /\.js|jsx$/,
		    			loader: 'jsx?harmony'
		    			}]
		    		}
	    }
    
在webpack.config.js文件当中，添加的modules包含了**jsx-loader**，也就是JSX加载器。如果在配置文件当中没有添加对应加载器的参数，后面在编译之后就会出现**You may need an appropriate loader to handle this file type.**相关的提醒。

* 在工程中的**components**文件夹中新建一个组件，例如**BlackBoard.js**.

		var React = require('react');
		var BlackBoard = React.createClass({
		render : function () {
			return (<div>
				Hello,Webpack,I am React!!
				</div>);
			}
		});
		module.exports = BlackBoard;

* 然后回到**main.js**文件引用该组件:

		var React = require('react');
		var ReactDOM = require('react-dom');
			var MessageBoard = require('../	component/BlackBoard.js');
			ReactDOM.render(<MessageBoard />,
			document.getElementById('container')
		)

* 最后回到**index.html**文件中，使用该组件:

		<!DOCTYPE html>
		<html lang="en">
		<head>
			<meta charset="UTF-8">
			<title>ReactWebpack</title>
		</head>
		<body>
			<div id="container"></div>
			<script src="bundle.js"></script>
		</body>
		</html>

cd到工程的根目录，执行:**webpack**进行编译打包。

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/webpack-command.png)

然后run当前工程,效果是:

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/webpack-result.png)

* 这里我发现只要我更改了我工程文件之后，需要重新执行**webpack**命令，从而进行编译输出。为了解决webpack可以**实时编译**问题，可以执行下面的方法:

	* 在工程根目录下执行:**webpack --watch**.
	* 在webpack.config.js最外层添加一个属性:**watch:true**.那么在第一次执行**webpack**命令之后，就会执行实时编译。

上面的就是react+webpack最简单的使用流程了。当然，这里只是对自定义的组件实现了**webpack**加载，那么，如果想对CSS文件，图片等可以实现同样的组件化吗？是的，**webpack**就是为了解决问题而来的。这里，我就会觉得**React + webpack 就是天作之合**。

好，同样地先安装对应的**loader**，执行:**npm install css-loader style-loader image-webpack-loader --save-dev**

同样地在**webpack.config.js**中添加相对应的loader脚本:
	
	 var path = require('path');
	module.exports = {
		entry: path.resolve(__dirname, 'app/main.js'),
		output: {
		path: path.join(__dirname, 'build'),
				filename: 'bundle.js'     },
	    module: {
	    loaders: [ {
		    test: /\.css$/,
		    loader: 'style-loader!css-loader'
	    },{
	    test: /\.(png|jpg)$/,
		    loaders: [
		    'file?hash=sha512&digest=hex&name=[hash].[ext]',
		    'image-webpack?bypassOnDebug&optimizationLevel=7&interlaced=false'
		    ]
	    },{
		    test: /\.js|jsx$/,
		    loader: 'jsx?harmony'
	    }]
	   }
    }


在**app**文件夹下添加一个文件,**boardStyle.css**:

	.BackgroundStyle{
		width:200px;
		height:100px;
		background-color: #40a070;
	}

再回到**main.js**引入这个CSS组件:
	
	 var React = require('react');
	var ReactDOM = require('react-dom');
	var MessageBoard = require('../component/BlackBoard.js');
		require('../app/boardStyle.css');//引入该组件
		ReactDOM.render(<MessageBoard />,
		document.getElementById('container')
	)

最后更改**index.html**文件，为当前组件添加一个样式:

	<div id="container" class="BackgroundStyle"></div>
	<script src="bundle.js"></script>

重新执行**webpack**(如果你没有开启了实时编译)，重新run一下工程。

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/webpack-result-CSS.png) 

到此，添加CSS组件流程完成了。如果需要插入图片的，使用同样的方法。但是要注意一点的是，根据当前
的配置，webpack会对图片进行进行一定的压缩处理。从而得到一个名字经过Hash之后的文件名字，此时，
我们可以保持引入图片的名字不变，但是路径执行压缩之后的图片的路径。这样就可以引用这个压缩后的图片了。

其他技巧:

* 我们引入一个组件或者文件的时候，需要指定它的所在路径，但是引入**React**这些组件的时候而不再使用添加具体的路径了。这个也让我产生了一些思考，后来我发现可以在**webpack.config.js**文件中添加
	一个字段:
	
		alias:[componentName : "component path",.....],
	
**componetnName:** 组件的引入名字，引入时候直接: **require（'componetnName')**;即可。
为什么要这样做呢？其实就是一些组件需要频繁地被引入，使用全局配置改组件的名称以及路径就可以减小
webpack查找组建的时间，效率更高一些。

###总结
好了，今天的React+webpack的tutorial全部完成了。接下来我打算继续学习一下Flux和Redux，然后去到awesome-react上面的开源项目学习。说到这些，突然我发现了ant.design组件库，没错，又要点爆我了。
哈哈，当初自己因为抱着**原生与JS交互**的问题不放，竟然走到了这里。
我是否应该走上前端开发之路呢，我感觉它同样很有趣呀。不如，keep curious,passion!

----

##20160228
## WebView与JavaScriptcore实践

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/javascriptCore.jpg) 

>记得一个月前一位前端的朋友问我关于JavaScript如何调用iOS原生方法的问题。我当时我也不知道如何做，就推荐了kitten同学的博客文章[UIWebView与JS的深度交互](http://kittenyang.com/webview-javascript-bridge /).不过最近我也开始学习前端开发了，回头看了一下这个问题，也把这个问题解决并总结一下。希望可以让你得到一定帮助。


先上源码:
[Object-C:](https://github.com/zhaiyjgithub/JavaScriptCore-Object-C.git)
[swift:](https://github.com/zhaiyjgithub/JavaScriptCore-swift.git)

### native端调用JS端
创建工程，添加JavaScriptCore.framework这个依赖库，并添加`@import JavaScriptCore;`包，或者`#import <JavaScriptCore/JavaScriptCore.h>`也可以。
在工程中先添加一个按钮`callJSFunctioinBtn`,并为这个按钮添加事件`clickCallJSFunctionBtn`。然后添加一个webView和一个HTML文件`index.html`,最后使用webView加载这个添加到工程中的HTML文件。
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UIButton * callJSFunctioinBtn = [[UIButton alloc] initWithFrame:CGRectMake(10, 40, 80, 44)];
    callJSFunctioinBtn.backgroundColor = [UIColor redColor];
    callJSFunctioinBtn.titleLabel.font = [UIFont systemFontOfSize:12.0f];
    [callJSFunctioinBtn setTitle:@"调用JS方法" forState:(UIControlStateNormal)];
    [callJSFunctioinBtn addTarget:self action:@selector(clickCallJSFunctionBtn:) forControlEvents:(UIControlEventTouchUpInside)];
    [self.view addSubview:callJSFunctioinBtn];
    
    CGRect webViewFrame = CGRectMake(0, 100, kWidth, kHeight - 100);
    UIWebView *webView = [[UIWebView alloc] initWithFrame:webViewFrame];
    webView.delegate = self;
    
    NSString* path = [[NSBundle mainBundle] pathForResource:@"index" ofType:@"html"];
    NSURL* url = [NSURL fileURLWithPath:path];
    NSURLRequest* request = [NSURLRequest requestWithURL:url] ;
    [webView loadRequest:request];
    [self.view addSubview:webView];
    self.webView = webView;
}
```
`callJSFunctioinBtn`按钮事件的方法：

```
- (void)clickCallJSFunctionBtn:(UIButton *)btn{
    [self.webView stringByEvaluatingJavaScriptFromString:@"callJSFunction()"];
}
```

关于本地HTML文件的内容。当前HTML文件中使用JavaScript标签谢了一个JavaScript的方法`callJSFunction`，当该方法被调用，就会在当前的webView弹出一个弹框，内容是：`原生调用JS方法成功！！`

```
	<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>hello</title>
</head>
<body>
	hello,I am webView!you can alert or write something here!
    <script type="text/javascript">
    	function callJSFunction () {
            <!--    如果调用成功就输出弹框        -->
    		 alert("原生调用JS方法成功！！");
    	}
	</script>
</body>
</html>
```

接下来，我们再回头看看`clickCallJSFunctionBtn`这个方法。在这个方法里面，我们可以通过`[self.webView stringByEvaluatingJavaScriptFromString:@"callJSFunction()"];`调用了JS的方法了。
	OK，我们来run一下当前的工程。
	![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js1.png)
	然后我们再点击一下红色的按钮，并留意模拟器的输出。
	![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js2.png)
	从第二张图片可以知道，的确弹出一个弹框，并且弹框的内容跟index.html的`callJSFunction`方法内容是一致的。因此，原生调用JavaScript方法成功了。
	
在这里，你很可能会问，如果我要在native端向JS端传递一个参数呢？OK，我们接下来就继续解决这个问题。
	首先，先回到index.html文件中，在脚本中添加一个方法

```
function callJSFunctionWithParam (param) {
            <!--  如果调用成功就会输出传递的参数内容          -->
            alert("your parma is: " + param)
    	}
 
```

上面的方法就是将传递过啦的参数通过弹窗的方式输出。
	继续再修改一下原生按钮的事件方法，传递一个`mary`这个字符串过去.
	
```
 [self.webView stringByEvaluatingJavaScriptFromString:@"callJSFunctionWithParam(\"mary\")"];
```

继续run一下工程，你会发现，的确输出了弹框并且成功把参数传递过去了。

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js3.png)

到这里，你会继续问，如果我想传递一个数组或者字典过去呢？如果按照之前的方法，将参数变成一个数组或者字典地址过去，run工程之后发现是出错的。你不可能将数组中的每个值拿出来再拼接成为一个字符串过去吧？OK，到这里我们应该使用今天文章标题提到的JavaScriptcore了。
	我们重新修改`clickCallJSFunctionBtn`方法的内容
	
```
	 NSArray * params = @[@"tony",@"zack",@"kson"];
    JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    [context setExceptionHandler:^(JSContext *context, JSValue *value) {
        NSLog(@"JS exception: %@", value);
    }];
    JSValue *jsFunction = context[@"callJSFunctionWithParam"];
    [jsFunction callWithArguments:@[params]];
```

上面代码的一些解释：

* 定义一个简单的数组，用于被传递的参数
* 获取当前webView的JavaScriptContext。是的，只可以通过KVC这个黑魔法来获取`key` = `documentView.webView.mainFrame.javaScriptContext`这个环境包含了当前webView定义的JavaScript方法。
* 为这个JScontext设置语法执行异常结果回调。如果JS语法错误，那么就会执行这个block回调，并提示一些语法错误信息让我们参考。
* 定义一个类型为JSValue的jsFunction，并在context[@"callJSFunctionWithParam"]为其赋值。`callJSFunctionWithParam`这个就是在当前webView环境定义的方法。什么是`JSValue`呢？点击查看这个它的定义:

>A JSValue is a reference to a value within the JavaScript object space of a
 JSVirtualMachine. All instances of JSValue originate from a JSContext and
 hold a strong reference to this JSContext.
 
从上面我们知道它在JVM环境中，它代表任何从JSContext获取的实例，无论整型，字符型，数组还是方法。跟上面那样的获取，返回值都是一个`JSValue`

* 最后，调用`callWithArguments`这个方法，用数组的方式将参数h包装起来并发送过去。点击进入该方法所在的头文件`JSValue.h`中，你发现很多关于方法生命，参数都是通过数组的方式包装过去的。要记得，`参数传递的数组要跟JS方法定义的列表保持一致的`。

继续修改index.html文件的传递参数的方法:

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>hello</title>
</head>
<body>
	hello,I am webView!you can alert or write something here!
    <script type="text/javascript">
    	function callJSFunction () {
            <!--    如果调用成功就输出弹框        -->
    		 alert("原生调用JS方法成功！！");
    	}
    	function callJSFunctionWithParam (param) {
            <!--  如果调用成功就会输出传递的参数内容          -->
            alert("your parma is: " + param[0] + " " + param[1] + "" + param[2])
    	}
	</script>
</body>
</html>
```

OK，继续运行这个工程并点击按钮看一下输出:

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js4.png)

yeah!的确输出我们想要的效果。

另外，在JSValue.h文件找到了一些`toArray`,`toString`...等方法，究竟有什么用呢？
	如果将index.html文件中JS方法添加返回值，类型假如是String类型。那么在native端的事件点击事件方法中修改为：
	
	```
	    JSValue * jsReturnValue =  [jsFunction callWithArguments:@[params]];
    NSLog(@"js return value:%@",[jsReturnValue toString]);
	```

上面提到的方法就是这个作用了。到此，native调用JS端方法也是完成了。
接下来我们开始继续解决JS端调用native端方法。

### JS端调用native端
我们要知道的是:

* 可以通过KVC方法获取当前webView的JavaScript环境并执行JavaScript环境定义的方法。
* 另外，还可以通过`注入`的方式往当前JavaScript环境写入一个方法，相当于在index.html文件中定义方法。

``` 
NSString *jsFunctionText =
    @"var injectFunction = function() {"
    "douctment.write(\"方法注入成功\")"
    "}";
    [context evaluateScript:jsFunctionText];
```
同样可以KVC从当前JSContext中获取当前方法并执行。

* 如何让JavaScript的环境中可以`见到(调用到)`原生端的定义的对象呢？答案就是当对象遵守了`JSExport`协议即可。


下面，我们定义一个`Student`对象。
`Student.h`文件

```
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#import <JavaScriptCore/JavaScriptCore.h>

@protocol StudentJS <JSExport>

- (void)takePhoto;

@end

@interface Student : NSObject<StudentJS>
@property(nonatomic,strong)UIViewController * viewController;

@end

```

* 上面定义了一个`StudentJS`协议，它遵循`JSExport`协议。并添加一个协议方法`takePhoto`.
* 然后下面这个`Student`对象遵循`StudentJS`协议。

然后在`Student.m`实现这个方法。

```
#import "Student.h"
#import "PhotoPickerTool.h"
#import "ViewController.h"

@implementation Student

- (void)takePhoto{
    NSLog(@"add a student");
    [[PhotoPickerTool sharedPhotoPickerTool] showOnPickerViewControllerSourceType:(UIImagePickerControllerSourceTypeSavedPhotosAlbum) onViewController:self.viewController compled:^(UIImage *image, NSDictionary *editingInfo) {
        NSLog(@"make photo");
        ViewController * VC =  (ViewController *)(self.viewController);
        VC.summerImageView.image = image;
    }];
}

@end

```

调用了这个方法就会打开摄像头拍照。

接下里然后在`viewDidLoad()`方法末尾添加一个`imageView`控件，用于显示拍照后的图片显示

```
    self.summerImageView = [[UIImageView alloc] initWithFrame:CGRectMake(120, 20, 80, 80)];
    [self.view addSubview:self.summerImageView];
```

回到index.html文件中，添加一个按钮，按钮的id=`pid`。需要知道的是在JavaScript中可以通过`id`的方式来获取`目的标签`并使用它。这个跟iOS端使用`tag`方式获取目的控件一样的原理。

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>hello</title>
</head>
<body>
	hello,I am webView!you can alert or write something here!
    <script type="text/javascript">
    	function callJSFunction () {
            <!--    如果调用成功就输出弹框        -->
    		 alert("原生调用JS方法成功！！");
    	}
    	function callJSFunctionWithParam (param) {
            <!--  如果调用成功就会输出传递的参数内容          -->
            alert("your parma is: " + param[0] + " " + param[1] + " " + param[2])
    	}
	</script>
    <!-- 添加一个按钮，id = "pid" -->
    <button id="pid">click me</button>
</body>
</html>
```

接下来回到`viewController.m`文件中添加代码:

```
- (void)webViewDidFinishLoad:(UIWebView *)webView{
    NSLog(@"webView finsh load");

    JSContext *context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    
    [context setExceptionHandler:^(JSContext *context, JSValue *value) {
        NSLog(@"WEB JS: %@", value);
    }];
    
    Student * kson = [[Student alloc] init];
    kson.viewController = self;
    context[@"myStudent"] = kson;
    
    NSString * str =
    @"function spring () {"
    "   myStudent.takePhoto();}"
    "var btn = document.getElementById(\"pid\");"
    "btn.addEventListener('click', spring);";
    
    [context evaluateScript:str];
    
}

```

* 首先获取当前webView的JavaScript环境
* 设置JS代码语法运行handler block
* 定义一个`Student`带对象，并将其`注册`到JavaScript环境中。因为`Student`对象已经遵守了`JSExport`协议，因此该类型对象在JavaScript环境是`可见(可以被访问)`。
* 定义一个JS的方法，名称叫`spring`。作用就是先调用上面注册的原生对象`myStudent`的方法`takePhoto()`.然后获取HTML文件中定义的按钮，并为这个按钮添加`点击`的监听事件，事件就是`spring`。最后将其拼接成为一个脚步字符串。
* 最后context将上面脚本同样注册到JS环境中。

接下来，run一下工程，可以看到webView上面已经多了一个title叫`click me`的按钮。

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js5.png)

然后点击`click me`按钮，就会弹出手机相册，选择其中任意一张图片后就会在原来界面顶部显示刚才选择的照片。

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js7.png)

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/js6.png)

同样地，如果要在JS端传值到native端。只需要在`takePhoto`这些遵循了`JSExport`协议的方法添加参数，然后在JS端的方法传递参数即可。你会发现，JS端的方法的变量定义跟`swfit`有些相似的。

OK，到此为止，JS端调用native端的问题同样也得到了解决了。不过，事情还没有到此结束的。当时我朋友跟我说，他的iOS程序猿是用`swift`编写项目的。刚开始我还以为很简单地转换一下就好了，最后发现的确是简单转换一下，但是还是遇到了一些`坎(并不是坑，因为我的swift语言基础仍然是很渣渣，所以是坎)`。

### swfit版

index.html文件内容跟OC测试环境一样。先看下webView的delegate方法

```
    func webViewDidFinishLoad(webView: UIWebView) {
        print("finshed load")
        let context = webView.valueForKeyPath("documentView.webView.mainFrame.javaScriptContext") as!  JSContext
        
        let kson = Student()
        kson.delegate = self
        
        context.setObject(kson, forKeyedSubscript:"Student")
        context.exceptionHandler = { context, exception in
            print("JS Error: \(exception)")
        }
        
        let script = "function spring () {document.write(\"kson\");Student.takePhoto();}var btn = document.getElementById(\"pid\");btn.addEventListener('click', spring);";
        context.evaluateScript(script)
    }
```

要修该的地方是，`context.setObject(kson, forKeyedSubscript:"Student")`,只能通过这样的方式来`注册native对象`

接下来也就是最重要的地方就是在`Student.h`文件中，`StudentJS`代理协议的定义，前面必须要加上`@Objc`，否则native对象在JavaScript环境中不可见。

```
@objc protocol StudentJS:JSExport{
    func takePhoto()
}
```

其他地方保持一致即可，继续运行工程，效果跟之前的一样。

###结束语
OK，关于webView与JavaScriptCore的实践过程到此结束。通过这一个总结，让自己对这个实践过程更加地深刻。同时，希望也可以给你带来一些帮助。好吧，接下来继续学习一下前端开发，努力再努力。


----

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


