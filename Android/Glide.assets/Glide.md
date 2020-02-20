[toc]



## Glide

### 基本概念：

Model：表示的是数据来源，可以是URL，本地文件或者是资源ID类型

Data：从数据源获取数据之后就会将其加工成原始数据；

Resource：对原始数据进行解码操作，解码之后的资源

TransformedResource：转换后的资源

TranscodeedResource：转码完成之后的资源

Target：将显示的目标封装为Target，显示在ImageView上



过程：

![image-20200219193553836](Glide.assets/image-20200219193553836.png)





### 源码解析



#### 使用方法：

```java
Glide.with(this).load(url).into(imageView);
```

##### 常用配置方式：

![image-20200219194147631](Glide.assets/image-20200219194147631.png)

with()方法中的参数的格式会决定给整个Glide加载图片的生命周期：

- 当传入的是Activity的时候，会随着活动的finish而终止
- application的时候，只有当应用程序被杀死或者是死机的时候才会被终止

Glide：可以限制图片的大小，

- fitCenter()：限制图片的最大长宽为ImageView的长款限制，但是不一定被填满
- centerCrop()：完全填充ImageView，但是会裁剪额外的图片
- diskCacheStrategy()：当图片的url变化的特别快的时候，可以直接跳过磁盘缓存
- priority()：并不能保证所有的图片都是按照这个优先级加载
- into()：指定显示图片的ImageView



### 源码解析：

```java
Glide.with(this).load(url).into(imageView);
```



#### with():Glide的静态方法

Gilde加载图片的过程与组件生命周期挂钩

重载了很多的with()方法：可以灵活的根据当前的上下文组件进行不同的图片加载操作()

例如：

```java
public static RequestManager with(Context context){
	RequestManagerRetriever retriever = RequestManagerRetriever.get();
	return retriver.get(context);
}

/*
//RequestManagerRetriever:主要作用是产生requestManager对象的
RequestManager:管理图片处理请求
*/
```

![image-20200219224306278](Glide.assets/image-20200219224306278.png)

![image-20200219224358159](Glide.assets/image-20200219224358159.png)

- getApplicationManager()（采用的**双重锁机制的单例模式**）：调用该方法的时候会获取唯一一个requestManager，然后让图片请求的处理



![qq_pic_merged_1582124009681](Glide.assets/qq_pic_merged_1582124009681-1582133267818.jpg)

getApplicationManager():

![image-20200219234517791](Glide.assets/image-20200219234517791.png)

返回一个RequestManager实例；



##### RequestManager:管理Glide请求图片的



###### get()方法:

- **Application类型**

  ```java
  Glide.with(getApplicationContext()).load(url).into(imageView);
  ```

  with() ——>get(context)

  ![image-20200219234328335](Glide.assets/image-20200219234328335.png)

  由于传递的是application对象，故而会返回一个requestManager对象

  当应用程序停止的时候，Glide图片加载也会随之停止

  

- **非Application类型**

  以Activity为例：

  ![image-20200219234729111](Glide.assets/image-20200219234729111.png)

  - 不在后台线程：

    I，获得一个FragmentManager对象

    II，通过调用fragmentGet()方法

    ![image-20200219235234230](Glide.assets/image-20200219235234230.png)

    

    ![image-20200220000920522](Glide.assets/image-20200220000920522.png)

    

    RequestManagerFragment通过lifecycle来进行Activity生命周期的观察，从而将Glide图片的加载与Activity的生命周期绑定在一起，从而在Acitivity的生命周期中管理GLide图片加载的灵活性

    

  



##### 总结

- 主要是获取requestManager对象，RequestManager通过RequestManagerRetriever(生产RequestManager的类来处理的)
- 由于Glide绑定了组件的生命周期，故而可以根据组件的不同生命周期来完成相应的处理





#### load(View view):

![image-20200220001843621](Glide.assets/image-20200220001843621.png)

从上图可以看出，load()**有很多重载方法，为了支持多种的图片来源：本地图片，URL等；**



##### DrawableTypeRequest()：

表示Glide()加载图片的所有请求



- **实现的建造者模式**，使用Builder.builder()方法来构建对象，内部实现了很多的方法：缩略图，尺寸，解码器，缓存，优先级，转换，裁剪等等，主要是完成对Glide初始化时参数的配置。支持链式调用

- 继承的是GenericRequestBuilder()(Glide参数配置的最终的一个父类)

  RequestTrancker:请求追踪器，负责跟踪整个图片请求的周期，也可以负责取消或者重启一些失败的图片请求

  



###### 两个重要方法：

- **asBitmap():**强制加载静态图片

  ![image-20200220010148502](Glide.assets/image-20200220010148502.png)

- **asGif():**强制加载Gif图片

![image-20200220010204821](Glide.assets/image-20200220010204821.png)

两者各自返回不同的类型的xxxTypeRequest对象。

###### **load(model):**

主要完成数据的初始化操作（属于GenericRequestBuilder类），从而获取DrawableTypeRequest对象

###### 总结：

主要功能是将图片转化成为Bitmap或者是Gif类型，方便后续加载



##### loadGeneric():

创建两个ModelLoader，然后通过调用optionsApplier.apply()方法创建DrawbleTypeRequest()对象（通过传入GenericRequestBuilder()中的成员变量）



#### 2.3	into():

该方法涉及的是安卓中的UI变化的一些方面，所以需要在主线程完成

##### 2.3.1	Request建立和begin()方法

![image-20200220011208107](Glide.assets/image-20200220011208107-1582133267818.png)

- 首先判断是否在主线程
- 判断传递的View是否为空，为空时抛出异常
- 对图片进行裁剪<font color = 'red'>**①**</font>
- 最后通过glide.buildImageViewTarget()方法将图片所要显示的空间封装成为Target对象，然后传递给into()方法<font color = 'red'>**②**</font>



 

<font color = 'red'>**①**</font>applyCenterCrop()：在其所属的类中为空实现，但是GenericRequestBuilder的子类会根据各自的需求来实现自己的逻辑，以DrawableRequestBuilder为例，

会调用DrawableRequestBuilder.centerCrop方法：

![image-20200220012032037](Glide.assets/image-20200220012032037.png)

- Transformation：裁剪之后的转化工作

- getDrawableCenterCrop()：通过调用父类GenericRequestBuilder中的transform()方法来完成一些变量的初始化操作

  ![image-20200220012347614](Glide.assets/image-20200220012347614.png)

  



<font color = 'red'>**②**</font>Target方法的描述：

**2.1	buildImageViewTarget():**

![image-20200220012909147](Glide.assets/image-20200220012909147.png)



- ImageView——>onLoadTarget():

  ![image-20200220013330608](Glide.assets/image-20200220013330608.png)

  或者是显示失败图片：![image-20200220013441249](Glide.assets/image-20200220013441249.png)

- setResource():定义为抽象：可以根据Target显示的图片的来源不同，实现类：

  ![image-20200220013731969](Glide.assets/image-20200220013731969.png)

**2.2   LifeCycleListener**

![image-20200220012931728](Glide.assets/image-20200220012931728.png)

当with()方法设置为ApplicationContext类型的参数的时候，会通过该接口中的三个方法来判断当前应用的状态，从而判断何时停止图片的加载。



**2.3 buildTarget():**返回一个所需要的Target对象

![image-20200220014153995](Glide.assets/image-20200220014153995.png)

- buildTarget()：通过class对象来判断Glide需要加载的图片类型并创建相应的XXXImageViewTaget对象(GlideDrawableImageViewTarget(),BitmapImageViewTarget(),DrawableImageViewTarget())



2.4	Gilde.into()的父类——>   GenericRequestBuilder.into()方法

![image-20200220131253674](Glide.assets/image-20200220131253674.png)

- 创建一个加载图片需要的Request，并绑定Target
- 在创建Request之前需要对该Target之前绑定的Request进行删除
- 发送Request，将其交给RequestTracker请求跟踪器，进行相应处理



<font color='red'>**2.4.1**</font> 	recycle()：完成初始化的工作

 ![](Glide.assets/image-20200220132500758.png))



- REQUEST_POOL.offer(this):当Request不用的时候会将此Request放回到请求池中以供复用



