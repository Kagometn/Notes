### Android的事件分发机制



![](https://upload-images.jianshu.io/upload_images/944365-e7baca065f885271.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 1.1 什么是事件：

了解事件的分发，首先需要了解什么是事件，这里的事件指的就是点击事件，当用户触摸屏幕的时候，将会产生点击事件(Touch事件)

Touch事件的相关细节(发生触摸的位置，时间等)被封装成MotionEvent对象



##### 1.1.1 MotionEvent事件类型：

![1574055655671](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1574055655671.png)

**事件序列：**	其实就是从手指触摸屏幕开始到离开屏幕所发生的一系列事件

> 注：一般情况下，事件列都是以`DOWN`事件开始、`UP`事件结束，中间有无数的MOVE事件，如下图：

![](https://upload-images.jianshu.io/upload_images/944365-79b1e86793514e99.png)

##### 1.1.2  什么是事件分发：

​	事件分发其实就是将点击事件传递到具体的某个View以及处理的过程叫做**事件分发**。

##### 1.1.3  事件在那些对象间传递，顺序？

Activity的UI界面由Activity,ViewGroup,View以及其派生类组成

​	

![](https://user-gold-cdn.xitu.io/2019/7/29/16c3b74f8063d008?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



![](https://upload-images.jianshu.io/upload_images/944365-02c588300f6ad741.png?imageMogr2/auto-orient/strip|imageView2/2/w/590/format/webp)



事件分发在这三个对象间进行传递

当点击事件发生后，事件先传到Activity，再传到ViewGroup，最终传到View。

###### 事件分发过程是由哪些方法写作完成：

![](https://upload-images.jianshu.io/upload_images/944365-7c6642f518ffa3d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/675/format/webp)

总结：

​	![](https://upload-images.jianshu.io/upload_images/944365-d0a7e6f3c2bbefcc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)





##### 事件分发有啥用？

​	默认情况下事件分发会按照由Activity到ViewGroup再到View的顺序进行分发，当我们不想View处理的时候，让ViewGroup处理，进行拦截。这些可以运用到处理滑动冲突。

例如：外部冲突和内部滑动方向不一致，当ScrollView嵌套Fragment，且Fragment内部有个竖向的ListView，当用户左右滑动时，要让外部的View拦截单击事件，当用户上下滑动时，要让内部的View的拦截点击事件。

![](https://user-gold-cdn.xitu.io/2019/7/29/16c3b7591e422816?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 1.1.4 事件分发涉及到的函数以及相应的作用

![1574062578938](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1574062578938.png)

- dispatchTouchEvent：用来进行事件分发，若事件能够传递到当前的View，则此方法一定会被调用

- onInterceptTouchEvent：在dispatchTouchEvent方法内被调用，用来判断是否拦截某个时间。若当前View拦截某个事件，则该方法不会再被调用，返回结果表示是否拦截当前事件，该方法只在ViewGroup中存在。

- onTouchEvent:用来处理点击事件，返回结果表示是否小号当前时间，，若小号，则在统一时间序列中，当前View无法再次接收到事件

  三个方法的伪代码：

  ```java
  public boolean dispatchTouchEvent(MotionEvent ev){
      boolean consume = false;
      if(onInterceptTouchEvent(ev)){
          consume = onTouchEvent(ev);
      }else{
          consume = child.dispatchTouchEvent(ev);
      }
      return consume;
  }
  ```

#### 2.1  Activity的事件分发机制

当一个点击事件发生的时候，事件最先传到Activity的dispatchTouchEvent()进行事件分发.



##### 2.1.1源码分析

```java
/**
  * 源码分析：Activity.dispatchTouchEvent（）
  */ 
    public boolean dispatchTouchEvent(MotionEvent ev) {

            // 一般事件列开始都是DOWN事件 = 按下事件，故此处基本是true
            if (ev.getAction() == MotionEvent.ACTION_DOWN) {

                onUserInteraction();
                // ->>分析1

            }

            // ->>分析2
            if (getWindow().superDispatchTouchEvent(ev)) {

                return true;
                // 若getWindow().superDispatchTouchEvent(ev)的返回true
                // 则Activity.dispatchTouchEvent（）就返回true，则方法结束。即 ：该点击事件停止往下传递 & 事件传递过程结束
                // 否则：继续往下调用Activity.onTouchEvent

            }
            // ->>分析4
            return onTouchEvent(ev);
        }


/**
  * 分析1：onUserInteraction()
  * 作用：实现屏保功能
  * 注：
  *    a. 该方法为空方法
  *    b. 当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
  */
      public void onUserInteraction() { 

      }
      // 回到最初的调用原处

/**
  * 分析2：getWindow().superDispatchTouchEvent(ev)
  * 说明：
  *     a. getWindow() = 获取Window类的对象
  *     b. Window类是抽象类，其唯一实现类 = PhoneWindow类；即此处的Window类对象 = PhoneWindow类对象
  *     、
  */
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return mDecor.superDispatchTouchEvent(event);
        // mDecor = 顶层View（DecorView）的实例对象
        // ->> 分析3
    }

/**
  * 分析3：mDecor.superDispatchTouchEvent(event)
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return super.dispatchTouchEvent(event);
        // 调用父类的方法 = ViewGroup的dispatchTouchEvent()
        // 即 将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制

    }
    // 回到最初的调用原处

/**
  * 分析4：Activity.onTouchEvent（）
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
  public boolean onTouchEvent(MotionEvent event) {

        // 当一个点击事件未被Activity下任何一个View接收 / 处理时
        // 应用场景：处理发生在Window边界外的触摸事件
        // ->> 分析5
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
        
        return false;
        // 即 只有在点击事件在Window边界外才会返回true，一般情况都返回false，分析完毕
    }

/**
  * 分析5：mWindow.shouldCloseOnTouch(this, event)
  */
    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    // 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
    if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
            && isOutOfBounds(context, event) && peekDecorView() != null) {
        return true;
    }
    return false;
    // 返回true：说明事件在边界外，即 消费事件
    // 返回false：未消费（默认）
}
// 回到分析4调用原处
```

##### 2.1.2 总结

- 当一个点击事件发生的时候，从Activity的事件分发开始(Activity.dispatchTouchEvent)

  ![](https://upload-images.jianshu.io/upload_images/944365-f8fda76bbdad7b96.png)

  - 方法总结

    ![](https://upload-images.jianshu.io/upload_images/944365-e186b0edcb590546.png)

  而至于viewGroup的dispatchTouchEvent()什么时候返回true/false？可以再2.2中参看。

#### 2.2ViewGroup事件的分发机制

从上面的Activity事件分发机制可知，viewGroup事件分发机制从dispatchTouchEvent()开始

##### 2.2.1 源码分析：

Android5.0之后，viewGroup.dispatchTouchEvent()的源码发生了变化但是原理没变，故分析的是Android5.0版本

```java
/**
  * 源码分析：ViewGroup.dispatchTouchEvent（）
  */ 
    public boolean dispatchTouchEvent(MotionEvent ev) { 

    ... // 仅贴出关键代码

        // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
            if (disallowIntercept || !onInterceptTouchEvent(ev)) {  

            // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，可通过调用requestDisallowInterceptTouchEvent（）修改
            // 判断值2： !onInterceptTouchEvent(ev) = 对onInterceptTouchEvent()返回值取反
                    // a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
                    // b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
                    // c. 关于onInterceptTouchEvent() ->>分析1

                ev.setAction(MotionEvent.ACTION_DOWN);  
                final int scrolledXInt = (int) scrolledXFloat;  
                final int scrolledYInt = (int) scrolledYFloat;  
                final View[] children = mChildren;  
                final int count = mChildrenCount;  

        // 重点分析2
            // 通过for循环，遍历了当前ViewGroup下的所有子View
            for (int i = count - 1; i >= 0; i--) {  
                final View child = children[i];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                    child.getHitRect(frame);  

                    // 判断当前遍历的View是不是正在点击的View，从而找到当前被点击的View
                    // 若是，则进入条件判断内部
                    if (frame.contains(scrolledXInt, scrolledYInt)) {  
                        final float xc = scrolledXFloat - child.mLeft;  
                        final float yc = scrolledYFloat - child.mTop;  
                        ev.setLocation(xc, yc);  
                        child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  

                        // 条件判断的内部调用了该View的dispatchTouchEvent()
                        // 即 实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
                        if (child.dispatchTouchEvent(ev))  { 

                        mMotionTarget = child;  
                        return true; 
                        // 调用子View的dispatchTouchEvent后是有返回值的
                        // 若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
                        // 于是给ViewGroup的dispatchTouchEvent（）直接返回了true，即直接跳出
                        // 即把ViewGroup的点击事件拦截掉

                                }  
                            }  
                        }  
                    }  
                }  
            }  
            boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||  
                    (action == MotionEvent.ACTION_CANCEL);  
            if (isUpOrCancel) {  
                mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;  
            }  
            final View target = mMotionTarget;  

        // 重点分析3
        // 若点击的是空白处（即无任何View接收事件） / 拦截事件（手动复写onInterceptTouchEvent（），从而让其返回true）
        if (target == null) {  
            ev.setLocation(xf, yf);  
            if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {  
                ev.setAction(MotionEvent.ACTION_CANCEL);  
                mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
            }  
            
            return super.dispatchTouchEvent(ev);
            // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
            // 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()，即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的View.dispatchTouchEvent（））
            // 此处需与上面区别：子View的dispatchTouchEvent（）
        } 

        ... 

}
/**
  * 分析1：ViewGroup.onInterceptTouchEvent()
  * 作用：是否拦截事件
  * 说明：
  *     a. 返回true = 拦截，即事件停止往下传递（需手动设置，即复写onInterceptTouchEvent（），从而让其返回true）
  *     b. 返回false = 不拦截（默认）
  */
  public boolean onInterceptTouchEvent(MotionEvent ev) {  
    
    return false;

  } 
  // 回到调用原处
```

##### 2.2.2.  总结

结论：Android事件分发总是**先传递到viewGroup，再传递到view**.

过程：当点击了某个控件的时候：

![](https://upload-images.jianshu.io/upload_images/944365-6ec2e864af7ffd37.png?imageMogr2/auto-orient/strip|imageView2/2/w/713/format/webp)

##### 2.2.3  核心方法归纳：

![](https://upload-images.jianshu.io/upload_images/944365-ff627fea1a2244ad.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

Demo详解：

布局如下：

![](https://upload-images.jianshu.io/upload_images/944365-b0bf3dd7ad41b335.png?imageMogr2/auto-orient/strip|imageView2/2/w/210/format/webp)

- 测试代码

布局文件：*activity_main.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/my_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:focusableInTouchMode="true"
    android:orientation="vertical">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮1" />

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮2" />

</LinearLayout>
```

核心代码：*MainActivity.java*

```java
/**
  * ViewGroup布局（myLayout）中有2个子View = 2个按钮
  */
    public class MainActivity extends AppCompatActivity {

    Button button1,button2;
    ViewGroup myLayout;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button1 = (Button)findViewById(R.id.button1);
        button2 = (Button)findViewById(R.id.button2);
        myLayout = (LinearLayout)findViewById(R.id.my_layout);

        // 1.为ViewGroup布局设置监听事件
        myLayout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了ViewGroup");
            }
        });

        // 2. 为按钮1设置监听事件
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了button1");
            }
        });

        // 3. 为按钮2设置监听事件
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了button2");
            }
        });

    }
}
```

- 结果测试

  

  ![img](https:////upload-images.jianshu.io/upload_images/944365-a9c45aa25d12b589.png?imageMogr2/auto-orient/strip|imageView2/2/w/987/format/webp)

  

从上面的测试代码结果发现：

- 点击Button时，执行Button.onClick()，但是viewGroupLayout注册的onTouch()不会执行

- 只有点击空白区域时，才会执行viewGroupLayout的onTouch()

- 结论：Button的onCLick()将事件消费掉了，因此事件不会再继续向下传递

  

#### 2.3View事件的分发机制

从2.2可以看出，View事件分发机制从dispatchTouchEvent()开始

##### 2.3.1源码分析

```csharp
/**
  * 源码分析：View.dispatchTouchEvent（）
  */
  public boolean dispatchTouchEvent(MotionEvent event) {  

        if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
                mOnTouchListener.onTouch(this, event)) {  
            return true;  
        } 
        return onTouchEvent(event);  
  }
  // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent()
  //     1. mOnTouchListener != null
  //     2. (mViewFlags & ENABLED_MASK) == ENABLED
  //     3. mOnTouchListener.onTouch(this, event)
  // 下面对这3个条件逐个分析


/**
  * 条件1：mOnTouchListener != null
  * 说明：mOnTouchListener变量在View.setOnTouchListener（）方法里赋值
  */
  public void setOnTouchListener(OnTouchListener l) { 

    mOnTouchListener = l;  
    // 即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值（不为空）
        
} 

/**
  * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
  * 说明：
  *     a. 该条件是判断当前点击的控件是否enable
  *     b. 由于很多View默认enable，故该条件恒定为true
  */

/**
  * 条件3：mOnTouchListener.onTouch(this, event)
  * 说明：即 回调控件注册Touch事件时的onTouch（）；需手动复写设置，具体如下（以按钮Button为例）
  */
    button.setOnTouchListener(new OnTouchListener() {  
        @Override  
        public boolean onTouch(View v, MotionEvent event) {  
     
            return false;  
        }  
    });
    // 若在onTouch（）返回true，就会让上述三个条件全部成立，从而使得View.dispatchTouchEvent（）直接返回true，事件分发结束
    // 若在onTouch（）返回false，就会使得上述三个条件不全部成立，从而使得View.dispatchTouchEvent（）中跳出If，执行onTouchEvent(event)
```

接下来，我们继续看：**onTouchEvent(event)**的源码分析

> 1. 详情请看注释
> 2.  `Android 5.0`后 `View.onTouchEvent()`源码发生了变化（更加复杂），但原理相同；
> 3. 本文为了让读者更好理解，所以采用`Android 5.0`前的版本

```csharp
/**
  * 源码分析：View.onTouchEvent（）
  */
  public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  

    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
         
        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  

    // 若该控件可点击，则进入switch判断中
    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  

                switch (event.getAction()) { 

                    // a. 若当前的事件 = 抬起View（主要分析）
                    case MotionEvent.ACTION_UP:  
                        boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  

                            ...// 经过种种判断，此处省略

                            // 执行performClick() ->>分析1
                            performClick();  
                            break;  

                    // b. 若当前的事件 = 按下View
                    case MotionEvent.ACTION_DOWN:  
                        if (mPendingCheckForTap == null) {  
                            mPendingCheckForTap = new CheckForTap();  
                        }  
                        mPrivateFlags |= PREPRESSED;  
                        mHasPerformedLongPress = false;  
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                        break;  

                    // c. 若当前的事件 = 结束事件（非人为原因）
                    case MotionEvent.ACTION_CANCEL:  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                        removeTapCallback();  
                        break;

                    // d. 若当前的事件 = 滑动View
                    case MotionEvent.ACTION_MOVE:  
                        final int x = (int) event.getX();  
                        final int y = (int) event.getY();  
        
                        int slop = mTouchSlop;  
                        if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                                (y < 0 - slop) || (y >= getHeight() + slop)) {  
                            // Outside button  
                            removeTapCallback();  
                            if ((mPrivateFlags & PRESSED) != 0) {  
                                // Remove any future long press/tap checks  
                                removeLongPressCallback();  
                                // Need to switch from pressed to not pressed  
                                mPrivateFlags &= ~PRESSED;  
                                refreshDrawableState();  
                            }  
                        }  
                        break;  
                }  
                // 若该控件可点击，就一定返回true
                return true;  
            }  
             // 若该控件不可点击，就一定返回false
            return false;  
        }

/**
  * 分析1：performClick（）
  */  
    public boolean performClick() {  

        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);  
            mOnClickListener.onClick(this);  
            return true;  
            // 只要我们通过setOnClickListener（）为控件View注册1个点击事件
            // 那么就会给mOnClickListener变量赋值（即不为空）
            // 则会往下回调onClick（） & performClick（）返回true
        }  
        return false;  
    }  
```

##### 2.3.2 总结

- 每当控件被点击时：



![img](https:////upload-images.jianshu.io/upload_images/944365-76ce9e8299386729.png?imageMogr2/auto-orient/strip|imageView2/2/w/590/format/webp)

示意图

> 注：`onTouch（）`的执行 先于  `onClick（）`

- 核心方法总结



![img](https:////upload-images.jianshu.io/upload_images/944365-762cf45f36858bbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

##### 2.3.3 Demo讲解

下面我将用`Demo`验证上述的结论

```csharp
/**
  * 结论验证1：在回调onTouch()里返回false
  */
   // 1. 通过OnTouchListener()复写onTouch()，从而手动设置返回false
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());
           
                return false;
            }
        });

    // 2. 通过 OnClickListener（）为控件设置点击事件，为mOnClickListener变量赋值（即不为空），从而往下回调onClick（）
    button.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }

        });

/**
  * 结论验证2：在回调onTouch()里返回true
  */
   // 1. 通过OnTouchListener()复写onTouch()，从而手动设置返回true
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());
           
                return true;
            }
        });

    // 2. 通过 OnClickListener（）为控件设置点击事件，为mOnClickListener变量赋值（即不为空）
    // 但由于dispatchTouchEvent（）返回true，即事件不再向下传递，故不调用onClick()）
    button.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }
            
        });
```

- 测试结果

  

  ![img](https:////upload-images.jianshu.io/upload_images/944365-97959093583369a0.png?imageMogr2/auto-orient/strip|imageView2/2/w/904/format/webp)

  示意图

#### 2.4 总结



![img](https:////upload-images.jianshu.io/upload_images/944365-eeebede55f55b040.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



#### 3,工作流程  总结

- 一个事件分发的工作流程总结，具体如下：

![](https://upload-images.jianshu.io/upload_images/944365-aea821bbb613c195.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

>
>
>左侧虚线：具备相关性 & 逐层返回



- 以角色为核心的图解说明

![](https://upload-images.jianshu.io/upload_images/944365-bccafd3ff8a880ff.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 以方法为核心的图解说明

  

![](https://upload-images.jianshu.io/upload_images/944365-9f340a39bdad520e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 4，核心方法总结

- 已知事件分发过程的核心方法为：dispatchTouchEvent()，onInterceptTouchEvent()和onYTouchEvent()

![](https://upload-images.jianshu.io/upload_images/944365-faaf73d0f3eb870f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 下面，我将结合总结的工作流程，再次详细讲解该3个方法

##### 4.1 dispatchTouchEvent()

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-4fbf11afa24b033b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-f8888a622c255648.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

- 返回情况说明

**情况1：默认**




![img](https:////upload-images.jianshu.io/upload_images/944365-a4582905b3972904.png?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

示意图





![img](https:////upload-images.jianshu.io/upload_images/944365-fee3555bdc6f9524.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

**情况2：返回true**




![img](https:////upload-images.jianshu.io/upload_images/944365-b00acc9f529f5393.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图





![img](https:////upload-images.jianshu.io/upload_images/944365-6822b1af27852dbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

**情况3：返回false**




![img](https:////upload-images.jianshu.io/upload_images/944365-962fc440d2fffe0b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图





![img](https:////upload-images.jianshu.io/upload_images/944365-0a9ae1bc05f432b9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

------

##### 4.2 onInterceptTouchEvent()

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-f4116863606e494e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 注：`Activity`、`View`都无该方法



![img](https:////upload-images.jianshu.io/upload_images/944365-28d0f5e7a1665148.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

- 返回情况说明

**情况1：true**



![img](https:////upload-images.jianshu.io/upload_images/944365-9a83aed00a8c0a54.png?imageMogr2/auto-orient/strip|imageView2/2/w/926/format/webp)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-6889eda6ebda8c40.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

**情况2：false（默认）**



![img](https:////upload-images.jianshu.io/upload_images/944365-ea06029e3176635f.png?imageMogr2/auto-orient/strip|imageView2/2/w/926/format/webp)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-299cfcbe3a9c9fd5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

------

##### 4.3 onTouchEvent()

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-b5f527027257b98e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-744f5b7e8d413562.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

- 返回情况说明

**情况1：返回true**



![img](https:////upload-images.jianshu.io/upload_images/944365-36af591c11a8e450.png?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-aef94c6c182353a9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

**情况2：返回false（default）**




![img](https:////upload-images.jianshu.io/upload_images/944365-efd21a46a9af808e.png?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

示意图





![img](https:////upload-images.jianshu.io/upload_images/944365-5da9fe2990f75d9c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

##### 4.4 三者关系

下面，我用一段伪代码来阐述上述3个方法的关系 & 事件传递规则

```java
/**
  * 点击事件产生后
  */ 
  // 步骤1：调用dispatchTouchEvent（）
  public boolean dispatchTouchEvent(MotionEvent ev) {

    boolean consume = false; //代表 是否会消费事件
    
    // 步骤2：判断是否拦截事件
    if (onInterceptTouchEvent(ev)) {
      // a. 若拦截，则将该事件交给当前View进行处理
      // 即调用onTouchEvent (）方法去处理点击事件
        consume = onTouchEvent (ev) ;

    } else {

      // b. 若不拦截，则将该事件传递到下层
      // 即 下层元素的dispatchTouchEvent（）就会被调用，重复上述过程
      // 直到点击事件被最终处理为止
      consume = child.dispatchTouchEvent (ev) ;
    }

    // 步骤3：最终返回通知 该事件是否被消费（接收 & 处理）
    return consume;

   }
```

------

#### 5. 常见的事件分发场景

下面，我将通过实例说明**常见的事件传递情况 & 流程**

##### 5.1 背景描述

- 讨论的布局如下：



![img](https:////upload-images.jianshu.io/upload_images/944365-e0f526dd1b5731be.png?imageMogr2/auto-orient/strip|imageView2/2/w/438/format/webp)

示意图

- 情景 

  1. 用户先触摸到屏幕上`View C`上的某个点（图中黄区）

  > `Action_DOWN`事件在此处产生

  1. 用户移动手指
  2. 最后离开屏幕

##### 5.2 一般的事件传递情况

一般的事件传递场景有：

- 默认情况
- 处理事件
- 拦截`DOWN`事件
- 拦截后续事件（`MOVE`、`UP`）

###### 场景1：默认

- 即不对控件里的方法（`dispatchTouchEvent()`、`onTouchEvent()`、`onInterceptTouchEvent()`）进行重写 或 更改返回值

- 那么调用的是这3个方法的默认实现：调用下层的方法 & 逐层返回

- 事件传递情况：（呈

  ```
  U
  ```

  型） 

  1. 从上往下调用dispatchTouchEvent()

  > Activity A ->> ViewGroup B ->> View C

  1. 从下往上调用onTouchEvent()

  > View C ->> ViewGroup B ->> Activity A



![img](https:////upload-images.jianshu.io/upload_images/944365-161a6e6fc8723248.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

> 注：虽然`ViewGroup B`的`onInterceptTouchEvent`（）对`DOWN`事件返回了`false`，但后续的事件`（MOVE、UP）`依然会传递给它的`onInterceptTouchEvent()`
>  这一点与`onTouchEvent（）`的行为是不一样的：不再传递 & 接收该事件列的其他事件

###### 场景2：处理事件

设`View C`希望处理该点击事件，即：设置`View C`为可点击的`（Clickable）` 或 复写其`onTouchEvent（）`返回`true`

> 最常见的：设置`Button`按钮来响应点击事件

事件传递情况：（如下图）

-  `DOWN`事件被传递给C的`onTouchEvent`方法，该方法返回`true`，表示处理该事件
- 因为`View C`正在处理该事件，那么`DOWN`事件将不再往上传递给ViewGroup B 和 `Activity A`的`onTouchEvent()`；
- 该事件列的其他事件`（Move、Up）`也将传递给`View C`的`onTouchEvent()` 



![img](https:////upload-images.jianshu.io/upload_images/944365-77e933eb44682777.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

> 会逐层往`dispatchTouchEvent()` 返回，最终事件分发结束

###### 场景3：拦截DOWN事件

假设`ViewGroup B`希望处理该点击事件，即`ViewGroup B`复写了`onInterceptTouchEvent()`返回`true`、`onTouchEvent()`返回`true`
 事件传递情况：（如下图）

- `DOWN`事件被传递给`ViewGroup B`的`onInterceptTouchEvent()`，该方法返回`true`，表示拦截该事件，即自己处理该事件（事件不再往下传递）
- 调用自身的`onTouchEvent()`处理事件（`DOWN`事件将不再往上传递给`Activity A`的`onTouchEvent()`）
- 该事件列的其他事件`（Move、Up）`将直接传递给`ViewGroup B`的`onTouchEvent()`

> 注：
>
> 1. 该事件列的其他事件`（Move、Up）`将不会再传递给`ViewGroup B`的`onInterceptTouchEvent`（）；因：该方法一旦返回一次`true`，就再也不会被调用
> 2. 逐层往`dispatchTouchEvent()` 返回，最终事件分发结束



![img](https:////upload-images.jianshu.io/upload_images/944365-a5e7cfed2cba02c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

示意图

###### 场景4：拦截DOWN的后续事件

**结论**

- 若 `ViewGroup` 拦截了一个半路的事件（如`MOVE`），该事件将会被系统变成一个`CANCEL`事件 & 传递给之前处理该事件的子`View`；
- 该事件不会再传递给`ViewGroup` 的`onTouchEvent()` 
- 只有再到来的事件才会传递到`ViewGroup`的`onTouchEvent()` 

**场景描述**
 `ViewGroup B` 无拦截`DOWN`事件（还是`View C`来处理`DOWN`事件），但它拦截了接下来的`MOVE`事件

> 即 `DOWN`事件传递到`View C`的`onTouchEvent（）`，返回了`true`

**实例讲解**

- 在后续到来的MOVE事件，`ViewGroup B` 的`onInterceptTouchEvent（）`返回`true`拦截该`MOVE`事件，但该事件并没有传递给`ViewGroup B` ；这个`MOVE`事件将会被系统变成一个`CANCEL`事件传递给`View C`的`onTouchEvent（）` 
- 后续又来了一个`MOVE`事件，该`MOVE`事件才会直接传递给`ViewGroup B` 的`onTouchEvent()` 

> 后续事件将直接传递给`ViewGroup B` 的`onTouchEvent()`处理，而不会再传递给`ViewGroup B` 的`onInterceptTouchEvent（）`，因该方法一旦返回一次true，就再也不会被调用了。

-  `View C`再也不会收到该事件列产生的后续事件



![img](https:////upload-images.jianshu.io/upload_images/944365-1599f532038686cd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

至此，关于`Android`常见的事件传递情况 & 流程已经讲解完毕。

------

#### 6. 额外知识

##### 6.1 Touch事件的后续事件（MOVE、UP）层级传递

- 若给控件注册了`Touch`事件，每次点击都会触发一系列`action`事件（ACTION_DOWN，ACTION_MOVE，ACTION_UP等）
- 当`dispatchTouchEvent（）`事件分发时，只有前一个事件（如ACTION_DOWN）返回true，才会收到后一个事件（ACTION_MOVE和ACTION_UP）

> 即如果在执行ACTION_DOWN时返回false，后面一系列的ACTION_MOVE、ACTION_UP事件都不会执行

从上面对事件分发机制分析知：

- dispatchTouchEvent()、 onTouchEvent() 消费事件、终结事件传递（返回true）
- 而onInterceptTouchEvent 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的ACTION_MOVE和ACTION_UP事件接收起到非常大的作用

> 请记住：接收了ACTION_DOWN事件的函数不一定能收到后续事件（ACTION_MOVE、ACTION_UP）

**这里给出ACTION_MOVE和ACTION_UP事件的传递结论**：

- 结论1
   若对象（Activity、ViewGroup、View）的dispatchTouchEvent()分发事件后消费了事件（返回true），那么收到ACTION_DOWN的函数也能收到ACTION_MOVE和ACTION_UP

> 黑线：ACTION_DOWN事件传递方向
>  红线：ACTION_MOVE 、 ACTION_UP事件传递方向



![img](https:////upload-images.jianshu.io/upload_images/944365-93d0b1496e9e6ca4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

流程讲解

- 结论2
   若对象（Activity、ViewGroup、View）的onTouchEvent()处理了事件（返回true），那么ACTION_MOVE、ACTION_UP的事件从上往下传到该`View`后就不再往下传递，而是直接传给自己的`onTouchEvent()`& 结束本次事件传递过程。

> 黑线：ACTION_DOWN事件传递方向
>  红线：ACTION_MOVE、ACTION_UP事件传递方向



![img](https:////upload-images.jianshu.io/upload_images/944365-9d639a0b9ebf7b4a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

流程讲解

##### 6.2 onTouch()和onTouchEvent()的区别

- 该2个方法都是在`View.dispatchTouchEvent（）`中调用
- 但`onTouch（）`优先于`onTouchEvent`执行；若手动复写在`onTouch（）`中返回`true`（即 将事件消费掉），将不会再执行`onTouchEvent（）` 

> 注：若1个控件不可点击（即非`enable`），那么给它注册`onTouch`事件将永远得不到执行，具体原因看如下代码

```csharp
// &&为短路与，即如果前面条件为false，将不再往下执行
//  故：onTouch（）能够得到执行需2个前提条件：
     // 1. mOnTouchListener的值不能为空
     // 2. 当前点击的控件必须是enable的
mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)

// 对于该类控件，若需监听它的touch事件，就必须通过在该控件中重写onTouchEvent（）来实现
```

------

#### 7. 总结

- 一个事件序列只能被一个View拦截且消耗，一旦一个元素拦截了某次事件，那么同一个书简序列的所有时间都会直接交给处理，因此同一事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View处理。
- 某一个View一旦决定拦截，那么这个事件序列都只能由他来处理，并且他的onInterceptTouchEvent不会再被调用。
- 某个View一旦开始处理事件，如果他不消耗ACTION_DOWN事件，那么统一事件序列中的其他事件都不会再交给他来处理，并且将事件重新交由他的父元素去处理，即父元素的onTouchEvent会被调用。
- 如果View不消耗ACTION_DOWN意外的其他事件，那么这个点击事件就会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续接收到后续的事件，最终这些消失的点击事件会传递给Activity处理
- ViewGroup默认不会拦截任何事件。Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false
- 与`Android`事件分发最相关的知识：**自定义View**系列文章
   [（1）自定义View基础 - 最易懂的自定义View原理系列](https://www.jianshu.com/p/146e5cec4863)
   [（2）自定义View Measure过程 - 最易懂的自定义View原理系列](https://www.jianshu.com/p/1dab927b2f36)
   [（3）自定义View Layout过程 - 最易懂的自定义View原理系列](https://www.jianshu.com/p/158736a2549d)
   [（4）自定义View Draw过程- 最易懂的自定义View原理系列](https://www.jianshu.com/p/95afeb7c8335)