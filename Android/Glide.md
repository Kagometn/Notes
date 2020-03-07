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

![2020-02-21 155138](Glide.assets/2020-02-21 155138.png)





<font color='red'>**2.4.1**</font> 	recycle()：完成初始化的工作

 ![](Glide.assets/image-20200220132500758.png))



- REQUEST_POOL.offer(this):当Request不用的时候会将此Request放回到请求池中以供复用



![IMG_20200221_162951](Glide.assets/IMG_20200221_162951.jpg)



 ![image-20200221164514794](Glide.assets/image-20200221164514794.png)

![image-20200221164612352](Glide.assets/image-20200221164612352.png)

![image-20200221164638307](Glide.assets/image-20200221164638307.png)

![image-20200221164706940](Glide.assets/image-20200221164706940.png)

 ![image-20200221164759250](Glide.assets/image-20200221164759250.png)

![image-20200221164829766](Glide.assets/image-20200221164829766.png)

 ![image-20200221164855382](Glide.assets/image-20200221164855382.png)

![image-20200221165001952](Glide.assets/image-20200221165001952.png)



**DataFetcher:**

![image-20200221163841092](Glide.assets/image-20200221163841092.png)

![image-20200221163905717](Glide.assets/image-20200221163905717.png)

- 从其实现类可以看出其主要的用途就是实现了不同形式的图片数据类型

![IMG_20200221_170300](Glide.assets/IMG_20200221_170300.jpg)



![image-20200221170510831](Glide.assets/image-20200221170510831.png)

![image-20200221170547947](Glide.assets/image-20200221170547947.png)







#### 



### Glide缓存简介

Glide的缓存设计可以说是非常先进的，考虑的场景也很周全。在缓存这一功能上，Glide又将它分成了两个模块，一个是内存缓存，一个是硬盘缓存。

这两个缓存模块的作用各不相同，**内存缓存的主要作用是防止应用重复将图片数据读取到内存当中，而硬盘缓存的主要作用是防止应用重复从网络或其他地方重复下载和读取数据。**

内存缓存和硬盘缓存的相互结合才构成了Glide极佳的图片缓存效果，那么接下来我们就分别来分析一下这两种缓存的使用方法以及它们的实现原理。

##### 缓存Key

既然是缓存功能，就必然会有用于进行缓存的Key。那么Glide的缓存Key是怎么生成的呢？我不得不说，Glide的缓存Key生成规则非常繁琐，决定缓存Key的参数竟然有10个之多。不过繁琐归繁琐，至少逻辑还是比较简单的，我们先来看一下Glide缓存Key的生成逻辑。

生成缓存Key的代码在Engine类的load()方法当中，这部分代码我们在上一篇文章当中已经分析过了，只不过当时忽略了缓存相关的内容，那么我们现在重新来看一下：

```JAVA
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());

        ...
    }

    ...
}

```

可以看到，这里在第11行调用了fetcher.getId()方法获得了一个id字符串，这个字符串也就是我们要加载的图片的唯一标识，比如说如果是一张网络上的图片的话，那么这个id就是这张图片的url地址。

接下来在第12行，将这个id连同着signature、width、height等等10个参数一起传入到EngineKeyFactory的buildKey()方法当中，从而构建出了一个EngineKey对象，这个EngineKey也就是Glide中的缓存Key了。

可见，决定缓存Key的条件非常多，即使你用override()方法改变了一下图片的width或者height，也会生成一个完全不同的缓存Key。

EngineKey类的源码大家有兴趣可以自己去看一下，其实主要就是重写了equals()和hashCode()方法，保证只有传入EngineKey的所有参数都相同的情况下才认为是同一个EngineKey对象，我就不在这里将源码贴出来了。

##### 内存缓存

有了缓存Key，接下来就可以开始进行缓存了，那么我们先从内存缓存看起。

首先你要知道，**默认情况下，Glide自动就是开启内存缓存的**。也就是说，**当我们使用Glide加载了一张图片之后，这张图片就会被缓存到内存当中，只要在它还没从内存中被清除之前，下次使用Glide再加载这张图片都会直接从内存当中读取，而不用重新从网络或硬盘上读取了，这样无疑就可以大幅度提升图片的加载效率**。比方说你在一个RecyclerView当中反复上下滑动，RecyclerView中只要是Glide加载过的图片都可以直接从内存当中迅速读取并展示出来，从而大大提升了用户体验。

而Glide最为人性化的是，你甚至不需要编写任何额外的代码就能自动享受到这个极为便利的内存缓存功能，因为Glide默认就已经将它开启了。

那么既然已经默认开启了这个功能，还有什么可讲的用法呢？只有一点，如果你有什么特殊的原因需要禁用内存缓存功能，Glide对此提供了接口：

```JAVA
Glide.with(this)
     .load(url)
     .skipMemoryCache(true)
     .into(imageView);
```

可以看到，只需要调用skipMemoryCache()方法并传入true，就表示禁用掉Glide的内存缓存功能。

没错，关于Glide内存缓存的用法就只有这么多，可以说是相当简单。但是我们不可能只停留在这么简单的层面上，接下来就让我们就通过阅读源码来分析一下Glide的内存缓存功能是如何实现的。

**其实说到内存缓存的实现，非常容易就让人想到LruCache算法**（Least Recently Used），也叫近期最少使用算法。**它的主要算法原理就是把最近使用的对象用强引用存储在LinkedHashMap中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。LruCache的用法也比较简单，我在 Android高效加载大图、多图解决方案，有效避免程序OOM 这篇文章当中有提到过它的用法**，感兴趣的朋友可以去参考一下。

那么不必多说，Glide内存缓存的实现自然也是使用的LruCache算法。不过除了LruCache算法之外，Glide还结合了一种弱引用的机制，共同完成了内存缓存功能，下面就让我们来通过源码分析一下。

首先回忆一下，在上一篇文章的第二步load()方法中，我们当时分析到了在loadGeneric()方法中会调用Glide.buildStreamModelLoader()方法来获取一个ModelLoader对象。当时没有再跟进到这个方法的里面再去分析，那么我们现在来看下它的源码：



```JAVA
public class Glide {

    public static <T, Y> ModelLoader<T, Y> buildModelLoader(Class<T> modelClass, Class<Y> resourceClass,
            Context context) {
         if (modelClass == null) {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
                Log.d(TAG, "Unable to load null model, setting placeholder only");
            }
            return null;
        }
        return Glide.get(context).getLoaderFactory().buildModelLoader(modelClass, resourceClass);
    }

    public static Glide get(Context context) {
        if (glide == null) {
            synchronized (Glide.class) {
                if (glide == null) {
                    Context applicationContext = context.getApplicationContext();
                    List<GlideModule> modules = new ManifestParser(applicationContext).parse();
                    GlideBuilder builder = new GlideBuilder(applicationContext);
                    for (GlideModule module : modules) {
                        module.applyOptions(applicationContext, builder);
                    }
                    glide = builder.createGlide();
                    for (GlideModule module : modules) {
                        module.registerComponents(applicationContext, glide);
                    }
                }
            }
        }
        return glide;
    }

    ...
}
```

这里我们还是只看关键，在第11行去构建ModelLoader对象的时候，先调用了一个Glide.get()方法，而这个方法就是关键。我们可以看到，get()方法中实现的是一个单例功能，而创建Glide对象则是在第24行调用GlideBuilder的createGlide()方法来创建的，那么我们跟到这个方法当中：


```JAVA
public class GlideBuilder {
    ...

    Glide createGlide() {
        if (sourceService == null) {
            final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
            sourceService = new FifoPriorityThreadPoolExecutor(cores);
        }
        if (diskCacheService == null) {
            diskCacheService = new FifoPriorityThreadPoolExecutor(1);
        }
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        if (bitmapPool == null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                int size = calculator.getBitmapPoolSize();
                bitmapPool = new LruBitmapPool(size);
            } else {
                bitmapPool = new BitmapPoolAdapter();
            }
        }
        if (memoryCache == null) {
            memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
        }
        if (diskCacheFactory == null) {
            diskCacheFactory = new InternalCacheDiskCacheFactory(context);
        }
        if (engine == null) {
            engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
        }
        if (decodeFormat == null) {
            decodeFormat = DecodeFormat.DEFAULT;
        }
        return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
    }
}
```

这里也就是构建Glide对象的地方了。那么观察第22行，你会发现这里new出了一个LruResourceCache，并把它赋值到了memoryCache这个对象上面。你没有猜错，这个就是Glide实现内存缓存所使用的LruCache对象了。不过我这里并不打算展开来讲LruCache算法的具体实现，如果你感兴趣的话可以自己研究一下它的源码。

现在创建好了LruResourceCache对象只能说是把准备工作做好了，接下来我们就一步步研究Glide中的内存缓存到底是如何实现的。

刚才在Engine的load()方法中我们已经看到了生成缓存Key的代码，而内存缓存的代码其实也是在这里实现的，那么我们重新来看一下Engine类load()方法的完整源码：

```JAVA
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());

        EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
        if (cached != null) {
            cb.onResourceReady(cached);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from cache", startTime, key);
            }
            return null;
        }

        EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
        if (active != null) {
            cb.onResourceReady(active);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from active resources", startTime, key);
            }
            return null;
        }

        EngineJob current = jobs.get(key);
        if (current != null) {
            current.addCallback(cb);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Added to existing load", startTime, key);
            }
            return new LoadStatus(cb, current);
        }

        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);

        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Started new load", startTime, key);
        }
        return new LoadStatus(cb, engineJob);
    }

    ...
}
```

可以看到，这里在第17行调用了loadFromCache()方法来获取缓存图片，如果获取到就直接调用cb.onResourceReady()方法进行回调。如果没有获取到，则会在第26行调用loadFromActiveResources()方法来获取缓存图片，获取到的话也直接进行回调。只有在两个方法都没有获取到缓存的情况下，才会继续向下执行，从而开启线程来加载图片。

也就是说，Glide的图片加载过程中会调用两个方法来获取内存缓存，loadFromCache()和loadFromActiveResources()。这两个方法中一个使用的就是LruCache算法，另一个使用的就是弱引用。我们来看一下它们的源码：



```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...

    private EngineResource<?> loadFromCache(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        }
        EngineResource<?> cached = getEngineResourceFromCache(key);
        if (cached != null) {
            cached.acquire();
            activeResources.put(key, new ResourceWeakReference(key, cached, getReferenceQueue()));
        }
        return cached;
    }

    private EngineResource<?> getEngineResourceFromCache(Key key) {
        Resource<?> cached = cache.remove(key);
        final EngineResource result;
        if (cached == null) {
            result = null;
        } else if (cached instanceof EngineResource) {
            result = (EngineResource) cached;
        } else {
            result = new EngineResource(cached, true /*isCacheable*/);
        }
        return result;
    }

    private EngineResource<?> loadFromActiveResources(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        }
        EngineResource<?> active = null;
        WeakReference<EngineResource<?>> activeRef = activeResources.get(key);
        if (activeRef != null) {
            active = activeRef.get();
            if (active != null) {
                active.acquire();
            } else {
                activeResources.remove(key);
            }
        }
        return active;
    }

    ...
}
```

在loadFromCache()方法的一开始，首先就判断了isMemoryCacheable是不是false，如果是false的话就直接返回null。这是什么意思呢？其实很简单，我们刚刚不是学了一个skipMemoryCache()方法吗？如果在这个方法中传入true，那么这里的isMemoryCacheable就会是false，表示内存缓存已被禁用。

我们继续住下看，接着调用了getEngineResourceFromCache()方法来获取缓存。在这个方法中，会使用缓存Key来从cache当中取值，而这里的cache对象就是在构建Glide对象时创建的LruResourceCache，那么说明这里其实使用的就是LruCache算法了。

但是呢，观察第22行，当我们从LruResourceCache中获取到缓存图片之后会将它从缓存中移除，然后在第16行将这个缓存图片存储到activeResources当中。activeResources就是一个弱引用的HashMap，用来缓存正在使用中的图片，我们可以看到，loadFromActiveResources()方法就是从activeResources这个HashMap当中取值的。使用activeResources来缓存正在使用中的图片，可以保护这些图片不会被LruCache算法回收掉。

好的，从内存缓存中读取数据的逻辑大概就是这些了。概括一下来说，就是如果能从内存缓存当中读取到要加载的图片，那么就直接进行回调，如果读取不到的话，才会开启线程执行后面的图片加载逻辑。

现在我们已经搞明白了内存缓存读取的原理，接下来的问题就是内存缓存是在哪里写入的呢？这里我们又要回顾一下上一篇文章中的内容了。还记不记得我们之前分析过，当图片加载完成之后，会在EngineJob当中通过Handler发送一条消息将执行逻辑切回到主线程当中，从而执行handleResultOnMainThread()方法。那么我们现在重新来看一下这个方法，代码如下所示：

```java
class EngineJob implements EngineRunnable.EngineRunnableManager {

    private final EngineResourceFactory engineResourceFactory;
    ...

    private void handleResultOnMainThread() {
        if (isCancelled) {
            resource.recycle();
            return;
        } else if (cbs.isEmpty()) {
            throw new IllegalStateException("Received a resource without any callbacks to notify");
        }
        engineResource = engineResourceFactory.build(resource, isCacheable);
        hasResource = true;
        engineResource.acquire();
        listener.onEngineJobComplete(key, engineResource);
        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                cb.onResourceReady(engineResource);
            }
        }
        engineResource.release();
    }

    static class EngineResourceFactory {
        public <R> EngineResource<R> build(Resource<R> resource, boolean isMemoryCacheable) {
            return new EngineResource<R>(resource, isMemoryCacheable);
        }
    }
    ...
}
```

 在第13行，这里通过EngineResourceFactory构建出了一个包含图片资源的EngineResource对象，然后会在第16行将这个对象回调到Engine的onEngineJobComplete()方法当中，如下所示： 

```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    @Override
    public void onEngineJobComplete(Key key, EngineResource<?> resource) {
        Util.assertMainThread();
        // A null resource indicates that the load failed, usually due to an exception.
        if (resource != null) {
            resource.setResourceListener(key, this);
            if (resource.isCacheable()) {
                activeResources.put(key, new ResourceWeakReference(key, resource, getReferenceQueue()));
            }
        }
        jobs.remove(key);
    }

    ...
}
```

现在就非常明显了，可以看到，在第13行，回调过来的EngineResource被put到了activeResources当中，也就是在这里写入的缓存。

那么这只是弱引用缓存，还有另外一种LruCache缓存是在哪里写入的呢？这就要介绍一下EngineResource中的一个引用机制了。观察刚才的handleResultOnMainThread()方法，在第15行和第19行有调用EngineResource的acquire()方法，在第23行有调用它的release()方法。其实，EngineResource是用一个acquired变量用来记录图片被引用的次数，调用acquire()方法会让变量加1，调用release()方法会让变量减1，代码如下所示：



```java
class EngineResource<Z> implements Resource<Z> {

    private int acquired;
    ...

    void acquire() {
        if (isRecycled) {
            throw new IllegalStateException("Cannot acquire a recycled resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call acquire on the main thread");
        }
        ++acquired;
    }

    void release() {
        if (acquired <= 0) {
            throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call release on the main thread");
        }
        if (--acquired == 0) {
            listener.onResourceReleased(key, this);
        }
    }
}
```

也就是说，当acquired变量大于0的时候，说明图片正在使用中，也就应该放到activeResources弱引用缓存当中。而经过release()之后，如果acquired变量等于0了，说明图片已经不再被使用了，那么此时会在第24行调用listener的onResourceReleased()方法来释放资源，这个listener就是Engine对象，我们来看下它的onResourceReleased()方法：



```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...    

    @Override
    public void onResourceReleased(Key cacheKey, EngineResource resource) {
        Util.assertMainThread();
        activeResources.remove(cacheKey);
        if (resource.isCacheable()) {
            cache.put(cacheKey, resource);
        } else {
            resourceRecycler.recycle(resource);
        }
    }

    ...
}
```

可以看到，这里首先会将缓存图片从activeResources中移除，然后再将它put到LruResourceCache当中。这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。

这就是Glide内存缓存的实现原理。

##### 硬盘缓存

接下来我们开始学习硬盘缓存方面的内容。

不知道你还记不记得，在本系列的第一篇文章中我们就使用过硬盘缓存的功能了。当时为了禁止Glide对图片进行硬盘缓存而使用了如下代码：


```java
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
```

调用diskCacheStrategy()方法并传入DiskCacheStrategy.NONE，就可以禁用掉Glide的硬盘缓存功能了。

这个diskCacheStrategy()方法基本上就是Glide硬盘缓存功能的一切，它可以接收四种参数：

DiskCacheStrategy.NONE： 表示不缓存任何内容。
DiskCacheStrategy.SOURCE： 表示只缓存原始图片。
DiskCacheStrategy.RESULT： 表示只缓存转换过后的图片（默认选项）。
DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
上面四种参数的解释本身并没有什么难理解的地方，但是有一个概念大家需要了解，就是当我们使用Glide去加载一张图片的时候，Glide默认并不会将原始图片展示出来，而是会对图片进行压缩和转换（我们会在后面学习这方面的内容）。总之就是经过种种一系列操作之后得到的图片，就叫转换过后的图片。而Glide默认情况下在硬盘缓存的就是转换过后的图片，我们通过调用diskCacheStrategy()方法则可以改变这一默认行为。

好的，关于Glide硬盘缓存的用法也就只有这么多，那么接下来还是老套路，我们通过阅读源码来分析一下，Glide的硬盘缓存功能是如何实现的。

首先，和内存缓存类似，硬盘缓存的实现也是使用的LruCache算法，而且Google还提供了一个现成的工具类DiskLruCache。我之前也专门写过一篇文章对这个DiskLruCache工具进行了比较全面的分析，感兴趣的朋友可以参考一下 Android DiskLruCache完全解析，硬盘缓存的最佳方案 。当然，Glide是使用的自己编写的DiskLruCache工具类，但是基本的实现原理都是差不多的。

接下来我们看一下Glide是在哪里读取硬盘缓存的。这里又需要回忆一下上篇文章中的内容了，**Glide开启线程来加载图片后会执行EngineRunnable的run()方法，run()方法中又会调用一个decode()方法，那么我们重新再来看一下这个decode()方法的源码：**

```java
private Resource<?> decode() throws Exception {
    if (isDecodingFromCache()) {
        return decodeFromCache();
    } else {
        return decodeFromSource();
    }
}
```

可以看到，这里会分为两种情况，**一种是调用decodeFromCache()方法从硬盘缓存当中读取图片，一种是调用decodeFromSource()来读取原始图片**。**默认情况下Glide会优先从缓存当中读取，只有缓存中不存在要读取的图片时，才会去读取原始图片**。那么我们现在来看一下decodeFromCache()方法的源码，如下所示：

```java
private Resource<?> decodeFromCache() throws Exception {
    Resource<?> result = null;
    try {
        result = decodeJob.decodeResultFromCache();
    } catch (Exception e) {
        if (Log.isLoggable(TAG, Log.DEBUG)) {
            Log.d(TAG, "Exception decoding result from cache: " + e);
        }
    }
    if (result == null) {
        result = decodeJob.decodeSourceFromCache();
    }
    return result;
}
```

可以看到，这里会先去调用DecodeJob的decodeResultFromCache()方法来获取缓存，如果获取不到，会再调用decodeSourceFromCache()方法获取缓存，这两个方法的区别其实就是DiskCacheStrategy.RESULT和DiskCacheStrategy.SOURCE这两个参数的区别，相信不需要我再做什么解释吧。

那么我们来看一下这两个方法的源码吧，如下所示：

```java
public Resource<Z> decodeResultFromCache() throws Exception {
    if (!diskCacheStrategy.cacheResult()) {
        return null;
    }
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = loadFromCache(resultKey);
    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    return result;
}

public Resource<Z> decodeSourceFromCache() throws Exception {
    if (!diskCacheStrategy.cacheSource()) {
        return null;
    }
    long startTime = LogTime.getLogTime();
    Resource<T> decoded = loadFromCache(resultKey.getOriginalKey());
    return transformEncodeAndTranscode(decoded);
}
```

可以看到，它们都是调用了loadFromCache()方法从缓存当中读取数据，如果是decodeResultFromCache()方法就直接将数据解码并返回，如果是decodeSourceFromCache()方法，还要调用一下transformEncodeAndTranscode()方法先将数据转换一下再解码并返回。

然而我们注意到，这两个方法中在调用loadFromCache()方法时传入的参数却不一样，一个传入的是resultKey，另外一个却又调用了resultKey的getOriginalKey()方法。这个其实非常好理解，刚才我们已经解释过了，Glide的缓存Key是由10个参数共同组成的，包括图片的width、height等等。但如果我们是缓存的原始图片，其实并不需要这么多的参数，因为不用对图片做任何的变化。那么我们来看一下getOriginalKey()方法的源码：

```java
public Key getOriginalKey() {
    if (originalKey == null) {
        originalKey = new OriginalKey(id, signature);
    }
    return originalKey;
}
```

可以看到，这里其实就是忽略了绝大部分的参数，只使用了id和signature这两个参数来构成缓存Key。而signature参数绝大多数情况下都是用不到的，因此基本上可以说就是由id（也就是图片url）来决定的Original缓存Key。

搞明白了这两种缓存Key的区别，那么接下来我们看一下loadFromCache()方法的源码吧：

```java
private Resource<T> loadFromCache(Key key) throws IOException {
    File cacheFile = diskCacheProvider.getDiskCache().get(key);
    if (cacheFile == null) {
        return null;
    }
    Resource<T> result = null;
    try {
        result = loadProvider.getCacheDecoder().decode(cacheFile, width, height);
    } finally {
        if (result == null) {
            diskCacheProvider.getDiskCache().delete(key);
        }
    }
    return result;
}
```

这个方法的逻辑非常简单，调用getDiskCache()方法获取到的就是Glide自己编写的DiskLruCache工具类的实例，然后调用它的get()方法并把缓存Key传入，就能得到硬盘缓存的文件了。如果文件为空就返回null，如果文件不为空则将它解码成Resource对象后返回即可。

这样我们就将硬盘缓存读取的源码分析完了，那么硬盘缓存又是在哪里写入的呢？趁热打铁我们赶快继续分析下去。

刚才已经分析过了，在没有缓存的情况下，会调用decodeFromSource()方法来读取原始图片。那么我们来看下这个方法：

```java
public Resource<Z> decodeFromSource() throws Exception {
    Resource<T> decoded = decodeSource();
    return transformEncodeAndTranscode(decoded);
}
```

 这个方法中只有两行代码，decodeSource()顾名思义是用来解析原图片的，而transformEncodeAndTranscode()则是用来对图片进行转换和转码的。我们先来看decodeSource()方法 

```java
private Resource<T> decodeSource() throws Exception {
    Resource<T> decoded = null;
    try {
        long startTime = LogTime.getLogTime();
        final A data = fetcher.loadData(priority);
        if (isCancelled) {
            return null;
        }
        decoded = decodeFromSourceData(data);
    } finally {
        fetcher.cleanup();
    }
    return decoded;
}

private Resource<T> decodeFromSourceData(A data) throws IOException {
    final Resource<T> decoded;
    if (diskCacheStrategy.cacheSource()) {
        decoded = cacheAndDecodeSourceData(data);
    } else {
        long startTime = LogTime.getLogTime();
        decoded = loadProvider.getSourceDecoder().decode(data, width, height);
    }
    return decoded;
}

private Resource<T> cacheAndDecodeSourceData(A data) throws IOException {
    long startTime = LogTime.getLogTime();
    SourceWriter<A> writer = new SourceWriter<A>(loadProvider.getSourceEncoder(), data);
    diskCacheProvider.getDiskCache().put(resultKey.getOriginalKey(), writer);
    startTime = LogTime.getLogTime();
    Resource<T> result = loadFromCache(resultKey.getOriginalKey());
    return result;
}
```

这里会在第5行先调用fetcher的loadData()方法读取图片数据，然后在第9行调用decodeFromSourceData()方法来对图片进行解码。接下来会在第18行先判断是否允许缓存原始图片，如果允许的话又会调用cacheAndDecodeSourceData()方法。而在这个方法中同样调用了getDiskCache()方法来获取DiskLruCache实例，接着调用它的put()方法就可以写入硬盘缓存了，注意原始图片的缓存Key是用的resultKey.getOriginalKey()。

好的，原始图片的缓存写入就是这么简单，接下来我们分析一下transformEncodeAndTranscode()方法的源码，来看看转换过后的图片缓存是怎么写入的。代码如下所示：

```java
private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = transform(decoded);
    writeTransformedToCache(transformed);
    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    return result;
}

private void writeTransformedToCache(Resource<T> transformed) {
    if (transformed == null || !diskCacheStrategy.cacheResult()) {
        return;
    }
    long startTime = LogTime.getLogTime();
    SourceWriter<Resource<T>> writer = new SourceWriter<Resource<T>>(loadProvider.getEncoder(), transformed);
    diskCacheProvider.getDiskCache().put(resultKey, writer);
}
```

这里的逻辑就更加简单明了了。先是在第3行调用transform()方法来对图片进行转换，然后在writeTransformedToCache()方法中将转换过后的图片写入到硬盘缓存中，调用的同样是DiskLruCache实例的put()方法，不过这里用的缓存Key是resultKey。

这样我们就将Glide硬盘缓存的实现原理也分析完了。虽然这些源码看上去如此的复杂，但是经过Glide出色的封装，使得我们只需要通过skipMemoryCache()和diskCacheStrategy()这两个方法就可以轻松自如地控制Glide的缓存功能了。

了解了Glide缓存的实现原理之后，接下来我们再来学习一些Glide缓存的高级技巧吧。

##### 高级技巧

虽说Glide将缓存功能高度封装之后，使得用法变得非常简单，但同时也带来了一些问题。

比如之前有一位群里的朋友就跟我说过，他们项目的图片资源都是存放在七牛云上面的，而七牛云为了对图片资源进行保护，会在图片url地址的基础之上再加上一个token参数。也就是说，一张图片的url地址可能会是如下格式：

```java
http://url.com/image.jpg?token=d9caa6e02c990b0a
```

但是接下来问题就来了，token作为一个验证身份的参数并不是一成不变的，很有可能时时刻刻都在变化。而如果token变了，那么图片的url也就跟着变了，图片url变了，缓存Key也就跟着变了。结果就造成了，明明是同一张图片，就因为token不断在改变，导致Glide的缓存功能完全失效了。

这其实是个挺棘手的问题，而且我相信绝对不仅仅是七牛云这一个个例，大家在使用Glide的时候很有可能都会遇到这个问题。

那么该如何解决这个问题呢？我们还是从源码的层面进行分析，首先再来看一下Glide生成缓存Key这部分的代码：

```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());

        ...
    }

    ...
}
```

来看一下第11行，刚才已经说过了，这个id其实就是图片的url地址。那么，这里是通过调用fetcher.getId()方法来获取的图片url地址，而我们在上一篇文章中已经知道了，fetcher就是HttpUrlFetcher的实例，我们就来看一下它的getId()方法的源码吧，如下所示：

```java
public class HttpUrlFetcher implements DataFetcher<InputStream> {

    private final GlideUrl glideUrl;
    ...

    public HttpUrlFetcher(GlideUrl glideUrl) {
        this(glideUrl, DEFAULT_CONNECTION_FACTORY);
    }

    HttpUrlFetcher(GlideUrl glideUrl, HttpUrlConnectionFactory connectionFactory) {
        this.glideUrl = glideUrl;
        this.connectionFactory = connectionFactory;
    }

    @Override
    public String getId() {
        return glideUrl.getCacheKey();
    }

    ...
}
```

可以看到，getId()方法中又调用了GlideUrl的getCacheKey()方法。那么这个GlideUrl对象是从哪里来的呢？其实就是我们在load()方法中传入的图片url地址，然后Glide在内部把这个url地址包装成了一个GlideUrl对象。

很明显，接下来我们就要看一下GlideUrl的getCacheKey()方法的源码了，如下所示：

```java
public class GlideUrl {

    private final URL url;
    private final String stringUrl;
    ...

    public GlideUrl(URL url) {
        this(url, Headers.DEFAULT);
    }

    public GlideUrl(String url) {
        this(url, Headers.DEFAULT);
    }

    public GlideUrl(URL url, Headers headers) {
        ...
        this.url = url;
        stringUrl = null;
    }

    public GlideUrl(String url, Headers headers) {
        ...
        this.stringUrl = url;
        this.url = null;
    }

    public String getCacheKey() {
        return stringUrl != null ? stringUrl : url.toString();
    }

    ...
}

```

这里我将代码稍微进行了一点简化，这样看上去更加简单明了。GlideUrl类的构造函数接收两种类型的参数，一种是url字符串，一种是URL对象。然后getCacheKey()方法中的判断逻辑非常简单，如果传入的是url字符串，那么就直接返回这个字符串本身，如果传入的是URL对象，那么就返回这个对象toString()后的结果。

其实看到这里，我相信大家已经猜到解决方案了，因为getCacheKey()方法中的逻辑太直白了，直接就是将图片的url地址进行返回来作为缓存Key的。那么其实我们只需要重写这个getCacheKey()方法，加入一些自己的逻辑判断，就能轻松解决掉刚才的问题了。

创建一个MyGlideUrl继承自GlideUrl，代码如下所示：

```java
public class MyGlideUrl extends GlideUrl {

    private String mUrl;

    public MyGlideUrl(String url) {
        super(url);
        mUrl = url;
    }

    @Override
    public String getCacheKey() {
        return mUrl.replace(findTokenParam(), "");
    }

    private String findTokenParam() {
        String tokenParam = "";
        int tokenKeyIndex = mUrl.indexOf("?token=") >= 0 ? mUrl.indexOf("?token=") : mUrl.indexOf("&token=");
        if (tokenKeyIndex != -1) {
            int nextAndIndex = mUrl.indexOf("&", tokenKeyIndex + 1);
            if (nextAndIndex != -1) {
                tokenParam = mUrl.substring(tokenKeyIndex + 1, nextAndIndex + 1);
            } else {
                tokenParam = mUrl.substring(tokenKeyIndex);
            }
        }
        return tokenParam;
    }

}
```

可以看到，这里我们重写了getCacheKey()方法，在里面加入了一段逻辑用于将图片url地址中token参数的这一部分移除掉。这样getCacheKey()方法得到的就是一个没有token参数的url地址，从而不管token怎么变化，最终Glide的缓存Key都是固定不变的了。

当然，定义好了MyGlideUrl，我们还得使用它才行，将加载图片的代码改成如下方式即可：

```java
Glide.with(this)
     .load(new MyGlideUrl(url))
     .into(imageView);
```

也就是说，我们需要在load()方法中传入这个自定义的MyGlideUrl对象，而不能再像之前那样直接传入url字符串了。不然的话Glide在内部还是会使用原始的GlideUrl类，而不是我们自定义的MyGlideUrl类。

这样我们就将这个棘手的缓存问题给解决掉了。