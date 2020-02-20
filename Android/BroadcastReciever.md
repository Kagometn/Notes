## BroadcastReciever

#### 1，什么是BroadcastReceive?

broadcast receiver 是一个用来响应系统范围内的广播的组件。 很多广播发自于系统本身。—例如, 通知屏幕已经被关闭、电池低电量、照片被拍下的广播。 应用程序也可以发起广播。—例如, 通知其它程序，一些数据被下载到了设备.当BroadcastReceive接收到系统发送的广播时就可以直接执行相对应的操作或者弹出一些界面与用户进行交互

通过继承BroadcastReceiver来实现广播的接收和操作 ，而且每一个广播通过`Intent` 对象来传递。




**广播可以分为两类:标准广播(Normal broadcasts)和有序广播(Ordered broadcasts)**

​	**标准广播:** **是一种完全异步执行的广播**,在广播发出以后几乎所有的广播接收器会在同一时刻接收到这条广播消息.之间没有先后顺序，效率比较高，且无法被拦截

​	**有序广播:** **是一种同步执行的广播**,在广播发出以后同时只能有一个广播接收器接收，优先级高的广播接收器会优先接收，当优先级高的广播接收器的onReceiver()方法运行结束后，广播才会继续传递，且前面的广播接收器，当这个广播接收器中的逻辑执行完毕以后,广播才会继续传递.所有此时的广播是有优先级的,优先级高的可以先接收到这个广播,并且还可以截断这个广播,**后面的广播接收器就不会接收到这个广播了**.



#### 作用

**监听/接收  应用App发出的广播消息，并作出响应**

**BroadcastReceiver** 是对发送出来的 **Broadcast** 进行**过滤、接受和响应**的组件。首先将要发送的消息和用于过滤的信息（Action，Category）装入一个 **Intent** 对象，然后通过调用 **Context.sendBroadcast()** 、 **sendOrderBroadcast()** 方法把 Intent 对象以广播形式发送出去。 广播发送出去后，所以已注册的 BroadcastReceiver 会检查注册时的 **IntentFilter** 是否与发送的 Intent 相匹配，若匹配则会调用 BroadcastReceiver 的 **onReceiver()** 方法

所以当我们定义一个 BroadcastReceiver 的时候，都需要实现 onReceiver() 方法。BroadcastReceiver 的生命周期很短，在执行 onReceiver() 方法时才有效，一旦执行完毕，该Receiver 的生命周期就结束了



> 从实现的角度来看，Android中的广播将广播的发送者和接受者极大程度上解耦，使得系统能够方便集成，更易扩展。



#### 应用场景

- Android不同组件间的通信(应用内以及不同应用间)

- 多线程通信

- 与Android系统在特定情况下的通信

  > 电话呼入，网络可用时

##### 具体实现流程要点粗略概括如下：

- 广播接收者BroadcastReceiver通过Binder机制向AMS(Android Manager Service)进行注册
- 广播发送者通过binder机制向AMS发送广播
- AMS查找符合相应条件(IntentFilter/Permission等)的BroadcastReceiver，将广播发送到BroadcastReceiver(一般情况下是Activity)相应的消息循环队列中
- 消息循环执行拿到此广播，回调BroadcastReceiver注册方式，具体实现上会有不同。但是总体流程大致如上

#### 实现原理

#### 一，静态注册

静态注册即在**清单文件**中为 **BroadcastReceiver** 进行注册，使用**< receiver >**标签声明，并在标签内用 **< intent-filter >** 标签设置过滤器。**这种形式的 BroadcastReceiver 的生命周期伴随着整个应用，如果这种方式处理的是系统广播，那么不管应用是否在运行，该广播接收器都能接收到该广播**

###### 1.1，发送标准广播

首先，继承BroadcastReciever类创建一个用于接收表针广播的Reciever，在onRecieve()方法中取出Intent传递来的字符串

```java
public class NormalReceiver extends BroadcastReceiver {

    private static final String TAG = "NormalReceiver";

    public NormalReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        String msg = intent.getStringExtra("Msg");
        Log.e(TAG, msg);
    }

}
```

在清单文件中声明的BroadcastReciever，必须包含值为**NORMAL_ACTION** 字符串的 **action** 属性，该广播接收器才能收到以下代码中发出的广播

发送标准广播调用的是 **sendBroadcast(Intent)** 方法

```java
public class MainActivity extends AppCompatActivity {

    private final String NORMAL_ACTION = "com.example.normal.receiver";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void sendBroadcast(View view) {
        Intent intent = new Intent(NORMAL_ACTION);
        intent.putExtra("Msg", "Hi");
        sendBroadcast(intent);
    }

}
```

在清单文件声明注册BroadcastReciever

```java
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver android:name=".NormalReceiver">
            <intent-filter>
                <action android:name="com.example.normal.receiver" />
            </intent-filter>
        </receiver>
        
    </application>
```

###### 1.2	发送有序广播

首先，继承BroadcastReciever类创建三个用于接收有序广播的Reciever,名字一次为命名为OrderReceiver_1、OrderReceiver_2、OrderReceiver_3。此外，既然 Receiver 在接收广播时存在先后顺序，那么 Receiver 除了能从发送广播使用的 Intent 接收数据外，**优先级高的 Receiver 也能在处理完操作后向优先级低的 Receiver 传送处理结果**



```java
public class OrderReceiver_1 extends BroadcastReceiver {

    private final String TAG = "OrderReceiver_1";

    public OrderReceiver_1() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "OrderReceiver_1被调用了");

        //取出Intent当中传递来的数据
        String msg = intent.getStringExtra("Msg");
        Log.e(TAG, "OrderReceiver_1接收到的值： " + msg);

        //向下一优先级的Receiver传递数据
        Bundle bundle = new Bundle();
        bundle.putString("Data", "（Hello）");
        setResultExtras(bundle);
    }
}
```



```java
public class OrderReceiver_2 extends BroadcastReceiver {

    private final String TAG = "OrderReceiver_2";

    public OrderReceiver_2() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "OrderReceiver_2被调用了");

        //取出上一优先级的Receiver传递来的数据
        String data = getResultExtras(true).getString("Data");
        Log.e(TAG, "从上一优先级的Receiver传递来的数据--" + data);

        //向下一优先级的Receiver传递数据
        Bundle bundle = new Bundle();
        bundle.putString("Data", "（叶应是叶）");
        setResultExtras(bundle);
    }
}
```



```java
public class OrderReceiver_3 extends BroadcastReceiver {

    private final String TAG = "OrderReceiver_3";

    public OrderReceiver_3() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "OrderReceiver_3被调用了");

        //取出上一优先级的Receiver传递来的数据
        String data = getResultExtras(true).getString("Data");
        Log.e(TAG, "从上一优先级的Receiver传递来的数据--" + data);
    }
}
```

在清单文件中对三个 Receiver 进行注册，指定相同的 **action** 属性值，Receiver 之间的优先级使用 **priority** 属性来判定，数值越大，优先级越高



```xml
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        
        <receiver android:name=".OrderReceiver_1">
            <intent-filter android:priority="100">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
        
        <receiver android:name=".OrderReceiver_2">
            <intent-filter android:priority="99">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
        
        <receiver android:name=".OrderReceiver_3">
            <intent-filter android:priority="98">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
        
    </application>
```

发送有序广播调用的是 **sendOrderedBroadcast(Intent , String)** 方法，String 参数值在自定义权限时使用，下边会有介绍



```java
public class MainActivity extends AppCompatActivity {

    private final String NORMAL_ACTION = "com.example.normal.receiver";

    private final String ORDER_ACTION = "com.example.order.receiver";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void sendBroadcast(View view) {
        Intent intent = new Intent(NORMAL_ACTION);
        intent.putExtra("Msg", "Hi");
        sendBroadcast(intent);
    }

    public void sendOrderBroadcast(View view) {
        Intent intent = new Intent(ORDER_ACTION);
        intent.putExtra("Msg", "Hi");
        sendOrderedBroadcast(intent, null);
    }

}
```

运行结果是



```undefined
02-20 22:52:30.135 6714-6714/com.example.zy.myapplication E/OrderReceiver_1: OrderReceiver_1被调用了
02-20 22:52:30.135 6714-6714/com.example.zy.myapplication E/OrderReceiver_1: OrderReceiver_1接收到的值： Hi
02-20 22:52:30.143 6714-6714/com.example.zy.myapplication E/OrderReceiver_2: OrderReceiver_2被调用了
02-20 22:52:30.143 6714-6714/com.example.zy.myapplication E/OrderReceiver_2: 从上一优先级的Receiver传递来的数据--（Hello）
02-20 22:52:30.150 6714-6714/com.example.zy.myapplication E/OrderReceiver_3: OrderReceiver_3被调用了
02-20 22:52:30.150 6714-6714/com.example.zy.myapplication E/OrderReceiver_3: 从上一优先级的Receiver传递来的数据--（叶应是叶）
```

可以看出 Receiver 接收广播时不仅因为“**priority**”属性存在先后顺序，且 Receiver 之间也能够传递数据

此外，BroadcastReceiver 也能调用 **abortBroadcast()** 方法截断广播，这样低优先级的广播接收器就无法接收到广播了

#### **二、动态注册**

动态注册 BroadcastReceiver 是在**代码中定义并设置好一个 IntentFilter 对象，然后在需要注册的地方调用 Context.registerReceiver() 方法，调用 Context.unregisterReceiver() 方法取消注册，此时就不需要在清单文件中注册 Receiver 了**

这里采用在 Service 中注册广播接收器的形式，分别在**注册广播接收器**、**取消注册广播接受器**和**接收到广播**时输出Log



```java
public class BroadcastService extends Service {

    private BroadcastReceiver receiver;

    private final String TAG = "BroadcastService";

    public BroadcastService() {
    }

    @Override
    public void onCreate() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(MainActivity.ACTION);
        receiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                Log.e(TAG, "BroadcastService接收到了广播");
            }
        };
        registerReceiver(receiver, intentFilter);
        Log.e(TAG, "BroadcastService注册了接收器");
        super.onCreate();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        unregisterReceiver(receiver);
        Log.e(TAG, "BroadcastService取消注册接收器");
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

}
```

提供启动服务，停止服务、发送广播的方法



```java
public class MainActivity extends AppCompatActivity {

    public final static String ACTION = "com.example.receiver";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void startService(View view) {
        Intent intent = new Intent(this, BroadcastService.class);
        startService(intent);
    }

    public void sendBroadcast(View view) {
        Intent intent = new Intent(ACTION);
        sendBroadcast(intent);
    }

    public void stopService(View view) {
        Intent intent = new Intent(this, BroadcastService.class);
        stopService(intent);
    }

}
```

运行结果如下所示



```undefined
02-20 23:55:20.967 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService注册了接收器
02-20 23:55:22.811 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:23.179 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:23.461 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:23.694 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:23.960 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:24.282 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:24.529 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService接收到了广播
02-20 23:55:24.916 22784-22784/com.example.zy.myapplication E/BroadcastService: BroadcastService取消注册接收器
```

#### **三、<font color = 'red'>本地广播</font>**

**之前发送和接收到的广播全都是属于系统全局广播，即发出的广播可以被其他应用接收到，而且也可以接收到其他应用发送出的广播，这样可能会有不安全因素**

**因此，在某些情况下可以采用本地广播机制，使用这个机制发出的广播只能在应用内部进行传递，而且广播接收器也只能接收本应用内自身发出的广播**

本地广播是使用 **LocalBroadcastManager** 来对广播进行管理

|                             函数                             |     作用     |
| :----------------------------------------------------------: | :----------: |
| LocalBroadcastManager.getInstance(this).registerReceiver(BroadcastReceiver, IntentFilter) | 注册Receiver |
| LocalBroadcastManager.getInstance(this).unregisterReceiver(BroadcastReceiver); | 注销Receiver |
| LocalBroadcastManager.getInstance(this).sendBroadcast(Intent) | 发送异步广播 |
| LocalBroadcastManager.getInstance(this).sendBroadcastSync(Intent) | 发送同步广播 |

首先，创建一个 BroadcastReceiver 用于接收本地广播



```java
public class LocalReceiver extends BroadcastReceiver {

    private final String TAG = "LocalReceiver";

    public LocalReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "接收到了本地广播");
    }
    
}
```

之后就是使用 LocalBroadcastManager 对 LocalReceiver 进行注册和解除注册了



```java
    private LocalBroadcastManager localBroadcastManager;

    private LocalReceiver localReceiver;

    private final String LOCAL_ACTION = "com.example.local.receiver";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        localBroadcastManager = LocalBroadcastManager.getInstance(this);
        localReceiver = new LocalReceiver();
        IntentFilter filter = new IntentFilter(LOCAL_ACTION);
        localBroadcastManager.registerReceiver(localReceiver, filter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(batteryReceiver);
        localBroadcastManager.unregisterReceiver(localReceiver);
    }
    
    public void sendLocalBroadcast(View view) {
        Intent intent = new Intent(LOCAL_ACTION);
        localBroadcastManager.sendBroadcast(intent);
    }
```

需要注意的是，**本地广播是无法通过静态注册的方式来接收的，因为静态注册广播主要是为了在程序未启动的情况下也能接收广播，而本地广播是应用自己发送的，此时应用肯定是启动的了**

#### **四、使用私有权限**

使用动态注册广播接收器存在一个问题，即系统内的任何应用均可监听并触发我们的 Receiver 。通常情况下我们是不希望如此的

解决办法之一是在清单文件中为 **< receiver >** 标签添加一个 **android:exported="false"** 属性，标明该 Receiver 仅限应用内部使用。这样，系统中的其他应用就无法接触到该 Receiver 了

此外，也可以选择创建自己的使用权限，即在清单文件中添加一个 **< permission >** 标签来声明自定义权限



```xml
    <permission
        android:name="com.example.permission.receiver"
        android:protectionLevel="signature" />
```

自定义权限时必须同时指定 **protectionLevel** 属性值，系统根据该属性值确定自定义权限的使用方式

|      属性值       |                           限定方式                           |
| :---------------: | :----------------------------------------------------------: |
|      normal       | 默认值。较低风险的权限，对其他应用，系统和用户来说风险最小。系统在安装应用时会自动批准授予应用该类型的权限，不要求用户明确批准（虽然用户在安装之前总是可以选择查看这些权限） |
|     dangerous     | 较高风险的权限，请求该类型权限的应用程序会访问用户私有数据或对设备进行控制，从而可能对用户造成负面影响。因为这种类型的许可引入了潜在风险，所以系统可能不会自动将其授予请求的应用。例如，系统可以向用户显示由应用请求的任何危险许可，并且在继续之前需要确认，或者可以采取一些其他方法来避免用户自动允许 |
|     signature     | 只有在请求该权限的应用与声明权限的应用使用相同的证书签名时，系统才会授予权限。如果证书匹配，系统会自动授予权限而不通知用户或要求用户的明确批准 |
| signatureOrSystem | 系统仅授予Android系统映像中与声明权限的应用使用相同的证书签名的应用。请避免使用此选项，“signature”级别足以满足大多数需求，“signatureOrSystem”权限用于某些特殊情况 |

首先，新建一个新的工程，在它的清单文件中创建一个自定义权限，并声明该权限。**protectionLevel** 属性值设为“**signature**”



```xml
    <permission
        android:name="com.example.permission.receiver"
        android:protectionLevel="signature" />

    <uses-permission android:name="com.example.permission.receiver" />
```

然后，发送含有该权限声明的 Broadcast 。这样，只有使用相同证书签名且声明该权限的应用才能接收到该 Broadcast 了



```java
    private final String PERMISSION_PRIVATE = "com.example.permission.receiver";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void sendPermissionBroadcast(View view) {
        sendBroadcast(new Intent("Hi"), PERMISSION_PRIVATE);
    }
```

回到之前的工程
 首先在清单文件中声明权限



```xml
<uses-permission android:name="com.example.permission.receiver" />
```

创建一个 BroadcastReceiver



```java
public class PermissionReceiver extends BroadcastReceiver {

    private final String TAG = "PermissionReceiver";

    public PermissionReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "接收到了私有权限广播");
    }
}
```

然后注册广播接收器。因为 Android Studio 在调试的时候会使用相同的证书为每个应用签名，所以，在之前新安装的App发送出广播后，PermissionReceiver 就会输出 Log 日志



```java
    private final String PERMISSION_PRIVATE = "com.example.permission.receiver";

    private PermissionReceiver permissionReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        IntentFilter intentFilter1 = new IntentFilter("Hi");
        permissionReceiver = new PermissionReceiver();
        registerReceiver(permissionReceiver, intentFilter1, PERMISSION_PRIVATE, null);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(permissionReceiver);
    }
```

#### **五、实战演练**

##### **5.1、监听网络状态变化**

首先需要一个用来监测当前网络状态的工具类



```csharp
/**
 * Created by 叶应是叶 on 2017/2/21.
 */

public class NetworkUtils {

    /**
     * 标记当前网络状态，分别是：移动数据、Wifi、未连接、网络状态已公布
     */
    public enum State {
        MOBILE, WIFI, UN_CONNECTED, PUBLISHED
    }

    /**
     * 为了避免因多次接收到广播反复提醒的情况而设置的标志位，用于缓存收到新的广播前的网络状态
     */
    private static State tempState;

    /**
     * 获取当前网络连接状态
     *
     * @param context Context
     * @return 网络状态
     */
    public static State getConnectState(Context context) {
        ConnectivityManager manager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = manager.getActiveNetworkInfo();
        State state = State.UN_CONNECTED;
        if (networkInfo != null && networkInfo.isAvailable()) {
            if (isMobileConnected(context)) {
                state = State.MOBILE;
            } else if (isWifiConnected(context)) {
                state = State.WIFI;
            }
        }
        if (state.equals(tempState)) {
            return State.PUBLISHED;
        }
        tempState = state;
        return state;
    }

    private static boolean isMobileConnected(Context context) {
        return isConnected(context, ConnectivityManager.TYPE_MOBILE);
    }

    private static boolean isWifiConnected(Context context) {
        return isConnected(context, ConnectivityManager.TYPE_WIFI);
    }

    private static boolean isConnected(Context context, int type) {
        //getAllNetworkInfo() 在 API 23 中被弃用
        //getAllNetworks() 在 API 21 中才添加
        ConnectivityManager manager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            NetworkInfo[] allNetworkInfo = manager.getAllNetworkInfo();
            for (NetworkInfo info : allNetworkInfo) {
                if (info.getType() == type) {
                    return info.isAvailable();
                }
            }
        } else {
            Network[] networks = manager.getAllNetworks();
            for (Network network : networks) {
                NetworkInfo networkInfo = manager.getNetworkInfo(network);
                if (networkInfo.getType() == type) {
                    return networkInfo.isAvailable();
                }
            }
        }
        return false;
    }

}
```

然后声明一个 BroadcastReceiver ，在onReceive() 方法中用Log输出当前网络状态



```java
public class NetworkReceiver extends BroadcastReceiver {

    private static final String TAG = "NetworkReceiver";

    public NetworkReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        switch (NetworkUtils.getConnectState(context)) {
            case MOBILE:
                Log.e(TAG, "当前连接了移动数据");
                break;
            case WIFI:
                Log.e(TAG, "当前连接了Wifi");
                break;
            case UN_CONNECTED:
                Log.e(TAG, "当前没有网络连接");
                break;
        }
    }

}
```

在清单文件中注册广播接收器，“**android.net.conn.CONNECTIVITY_CHANGE**”是系统预定义好的 **action** 值，只要系统网络状态发生变化，NetworkReceiver 就能收到广播



```xml
        <receiver android:name=".NetworkReceiver">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
        </receiver>
```

此外，还要申请查看网络状态的权限



```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

##### **5.2、监听电量变化**

因为系统规定监听电量变化的广播接收器不能静态注册，所以这里只能使用动态注册的方式了



```java
private final String TAG = "MainActivity";

    private BroadcastReceiver batteryReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Intent.ACTION_BATTERY_CHANGED);
        batteryReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // 当前电量
                int currentBattery = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, 0);
                // 总电量
                int totalBattery = intent.getIntExtra(BatteryManager.EXTRA_SCALE, 0);         
                Log.e(TAG, "当前电量:" + currentBattery + "-总电量：" + totalBattery);
            }
        };
        registerReceiver(batteryReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(batteryReceiver);
    }
```

在 **onReceive(Context , Intent )** 中的 Intent 值包含了一些额外信息，可以取出当前电量和总电量

为了方便查看电量变化，可以在模拟器的“extended controls”面板中主动地改变模拟器的电量，查看Log输出



![img](https:////upload-images.jianshu.io/upload_images/2552605-dbe03d95cc3fb423?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

这里写图片描述

##### **5.3、应用安装更新卸载监听**

首先，创建 BroadcastReceiver



```java
public class AppReceiver extends BroadcastReceiver {

    private final String TAG = "AppReceiver";

    public AppReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        //判断广播类型
        String action = intent.getAction();
        //获取包名
        Uri appName = intent.getData();
        if (Intent.ACTION_PACKAGE_ADDED.equals(action)) {
            Log.e(TAG, "安装了：" + appName);
        } else if (Intent.ACTION_PACKAGE_REPLACED.equals(action)) {
            Log.e(TAG, "更新了：" + appName);
        } else if (Intent.ACTION_PACKAGE_REMOVED.equals(action)) {
            Log.e(TAG, "卸载了：" + appName);
        }
    }
}
```

注册广播接收器



```xml
<receiver android:name=".train.AppReceiver">
            <intent-filter>
                <!--安装应用-->
                <action android:name="android.intent.action.PACKAGE_ADDED" />
                <!--更新应用-->
                <action android:name="android.intent.action.PACKAGE_REPLACED" />
                <!--卸载应用-->
                <action android:name="android.intent.action.PACKAGE_REMOVED" />
                <!--携带包名-->
                <data android:scheme="package" />
            </intent-filter>
        </receiver>
```

#### 六，实现原理

#### 6.1	采用的模型

- Android中的光膜使用了设计模式中的**观察者模式**：基于消息的发布/订阅事件模型

  > 因此，Android将广播的发送者和接收者解耦，使得系统方便集成，更易扩展



#### 6.2	模型讲解

- 模型中有三个角色：

  - 消息订阅者(广播接收者)

  - 消息发布者(广播发布者)

  - 消息中心（AMS，即Activity Manager Service）

    

- 示意图&原理如下：

![示意图](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS05ZmNhOWZkMzk3OGNlZjEwLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

#### 6.3	使用流程

如下：

![示意图](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS03YzlmZjY1NmViZDFiOTgxLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)



##### 6.3.1	自定义广播接收者BroadcastReceiver

- 继承BroadcastReceiver基类
- 必须覆写抽象方法onReceiver()方法



>1，广播接收器收到相应的广播后，会自动回调onReceiver()方法
>
>2，一般情况下，onReceive方法会涉及到与其他组件之间的交互，如发送Notification,启动Service等
>
>3，默认情况下，广播接收器运行在UI线程，因此oNReceive()方法不能执行耗时操作，否则会导致ANR

- 代码范例

  

```java
/ 继承BroadcastReceivre基类
public class mBroadcastReceiver extends BroadcastReceiver {

  // 复写onReceive()方法
  // 接收到广播后，则自动调用该方法
  @Override
  public void onReceive(Context context, Intent intent) {
   //写入接收广播后的操作
    }
}

```

##### 6.3	广播接收器注册

注册的方式分为两种：静态注册，动态注册

###### 6.3.1	静态注册

- 注册方式：在AndroidManifest里通过***标签声明
- 属性说明：

```java
<receiver 
    android:enabled=["true" | "false"]
//此broadcastReceiver能否接收其他App的发出的广播
//默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
//继承BroadcastReceiver子类的类名
    android:name=".mBroadcastReceiver"
//具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
    android:permission="string"
//BroadcastReceiver运行所处的进程
//默认为app的进程，可以指定独立的进程
//注：Android四大基本组件都可以通过此属性指定自己的独立进程
    android:process="string" >

//用于指定此广播接收器将接收的广播类型
//本示例中给出的是用于接收网络状态改变时发出的广播
 <intent-filter>
<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
<receiver 
    android:enabled=["true" | "false"]
//此broadcastReceiver能否接收其他App的发出的广播
//默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
//继承BroadcastReceiver子类的类名
    android:name=".mBroadcastReceiver"
//具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
    android:permission="string"
//BroadcastReceiver运行所处的进程
//默认为app的进程，可以指定独立的进程
//注：Android四大基本组件都可以通过此属性指定自己的独立进程
    android:process="string" >

//用于指定此广播接收器将接收的广播类型
//本示例中给出的是用于接收网络状态改变时发出的广播
 <intent-filter>
<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>

```

- 注册示例

  

```java
<receiver 
    //此广播接收者类是mBroadcastReceiver
    android:name=".mBroadcastReceiver" >
    //用于接收网络状态改变时发出的广播
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>

```

当APP首次启动的时候，系统会自动实例化mBroadcastReciever类，并注册到系统中

###### 6.3.2	动态注册

- 注册方式：在代码中调用Context.registerReciever()方法

- 具体代码如下：

  ```java
  // 选择在Activity生命周期方法中的onResume()中注册
  @Override
    protected void onResume(){
        super.onResume();
  
      // 1. 实例化BroadcastReceiver子类 &  IntentFilter
       mBroadcastReceiver mBroadcastReceiver = new mBroadcastReceiver();
       IntentFilter intentFilter = new IntentFilter();
  
      // 2. 设置接收广播的类型
      intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
  
      // 3. 动态注册：调用Context的registerReceiver（）方法
       registerReceiver(mBroadcastReceiver, intentFilter);
   }
  
  
  // 注册广播后，要在相应位置记得销毁广播
  // 即在onPause() 中unregisterReceiver(mBroadcastReceiver)
  // 当此Activity实例化时，会动态将MyBroadcastReceiver注册到系统中
  // 当此Activity销毁时，动态注册的MyBroadcastReceiver将不再接收到相应的广播。
   @Override
   protected void onPause() {
       super.onPause();
        //销毁在onResume()方法中的广播
       unregisterReceiver(mBroadcastReceiver);
       }
  }
  
  ```

  

###### 特别注意：

- **动态广播最好在Activity的onResume()注册，onPause()注销**

- **原因：**

  - **1，对于动态广播，有注册就必然的有注销，否则会造成内存泄漏**

    > **重复注册，重复注销也不允许**

  - Activity生命周期如下：

    

![Activity生命周期](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0zNTgyMDBmMTAwZWRmNzUzLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

Activity生命周期的方法是成对出现的：

- onCreate() & onDestory()
- onStart() & onStop()
- onResume() & onPause()

在onResume()注册、onPause()注销是因为onPause()在App死亡前一定会被执行，从而保证广播在App死亡前一定会被注销，从而防止内存泄露。

不在onCreate() & onDestory() 或 onStart() & onStop()注册、注销是因为：
**当系统因为内存不足（优先级更高的应用需要内存，请看上图红框）要回收Activity占用的资源时，Activity在执行完onPause()方法后就会被销毁，有些生命周期方法onStop()，onDestory()就不会执行。当再回到此Activity时，是从onCreate方法开始执行**。
假设我们将广播的注销放在onStop()，onDestory()方法里的话，有可能在Activity被销毁后还未执行onStop()，onDestory()方法，即广播仍还未注销，从而导致内存泄露。
但是，onPause()一定会被执行，从而保证了广播在App死亡前一定会被注销，从而防止内存泄露。

###### 6.3.3	两种注册方式的区别

![image-20200215174231329](C:\Users\Lenovo\Desktop\Android\image-20200215174231329.png)



#### 6.4	广播发送者向AMS发送广播

##### 6.4.1	广播的发送

广播 是 用”意图（Intent）“标识
定义广播的本质 = 定义广播所具备的“意图（Intent）”
广播发送 = 广播发送者 将此广播的“意图（Intent）”通过sendBroadcast（）方法发送出去

##### 6.4.2 广播的类型

广播的类型主要分为5类：

- 普通广播（Normal Broadcast）

- 系统广播（System Broadcast）

- 有序广播（Ordered Broadcast）

- 粘性广播（Sticky Broadcast）

- App应用内广播（Local Broadcast）

  具体说明如下：

###### 1，普通广播（Normal Broadcast）

即 开发者自身定义 intent的广播（最常用）。以context.sendBroadcast_"AsUser"(intent, ...)形式。

具体可以使用的方法有：

- sendBroadcast(intent)/sendBroadcast(intent

- receiverPermission)/sendBroadcastAsUser(intent, userHandler)/

  sendBroadcastAsUser(intent,userHandler,receiverPermission)。
  普通广播会被注册了的相应的感兴趣（intent-filter匹配）接收，且顺序是无序的。如果发送广播时有相应的权限要求，BroadCastReceiver如果想要接收此广播，也需要有相应的权限。

发送广播使用如下：

```java
Intent intent = new Intent();
//对应BroadcastReceiver中intentFilter的action
intent.setAction(BROADCAST_ACTION);
//发送广播
sendBroadcast(intent);
```

若被注册了的广播接收者中注册时intentFilter的action与上述匹配，则会接收此广播（即进行回调

onReceive()）。如下mBroadcastReceiver则会接收上述广播

```java
<receiver 
    //此广播接收者类是mBroadcastReceiver
    android:name=".mBroadcastReceiver" >
    //用于接收网络状态改变时发出的广播
    <intent-filter>
        <action android:name="BROADCAST_ACTION" />
    </intent-filter>
</receiver>
```

若发送广播有相应权限，那么广播接收者也需要相应权限

###### 2，系统广播（System Broadcast）

Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播
每个广播都有特定的Intent - Filter（包括具体的action），系统广播发出后，将相应的BroadcastReceiver接收，系统广播在系统内部当特定事件发生时，由系统自动发出

1. Android系统广播action如下：
   系统操作	action
   监听网络变化	android.net.conn.CONNECTIVITY_CHANGE
   关闭或打开飞行模式	Intent.ACTION_AIRPLANE_MODE_CHANGED
   充电时或电量发生变化	Intent.ACTION_BATTERY_CHANGED
   电池电量低	Intent.ACTION_BATTERY_LOW
   电池电量充足（即从电量低变化到饱满时会发出广播	Intent.ACTION_BATTERY_OKAY
   系统启动完成后(仅广播一次)	Intent.ACTION_BOOT_COMPLETED
   按下照相时的拍照按键(硬件按键)时	Intent.ACTION_CAMERA_BUTTON
   屏幕锁屏	Intent.ACTION_CLOSE_SYSTEM_DIALOGS
   设备当前设置被改变时(界面语言、设备方向等)	Intent.ACTION_CONFIGURATION_CHANGED
   插入耳机时	Intent.ACTION_HEADSET_PLUG
   未正确移除SD卡但已取出来时(正确移除方法:设置–SD卡和设备内存–卸载SD卡)	Intent.ACTION_MEDIA_BAD_REMOVAL
   插入外部储存装置（如SD卡）	Intent.ACTION_MEDIA_CHECKING
   成功安装APK	Intent.ACTION_PACKAGE_ADDED
   成功删除APK	Intent.ACTION_PACKAGE_REMOVED
   重启设备	Intent.ACTION_REBOOT
   屏幕被关闭	Intent.ACTION_SCREEN_OFF
   屏幕被打开	Intent.ACTION_SCREEN_ON
   关闭系统时	Intent.ACTION_SHUTDOWN
   重启设备	Intent.ACTION_REBOOT
   注：**当使用系统广播时，只需要在注册广播接收者时定义相关的action即可，并不需要手动发送广播，当系统有相关操作时会自动进行系统广播**

###### 3，有序广播（Ordered Broadcast）

**定义**

有序广播中的有序针对的是"广播接收者"，发送出去的广播被广播接收者按照先后顺序接收


广播接受者接收广播的顺序规则（同时面向静态和动态注册的广播接受者）按照Priority属性值从大-小排序；Priority属性相同者，动态注册的广播优先；
**特点**

1>多个具当前已经注册且有效的BroadcastReceiver接收有序广播时，是按照先后顺序接收的，先后顺序判定标准遵循为：**将当前系统中所有有效的动态注册和静态注册的BroadcastReceiver按照priority属性值从大到小排序，对于具有相同的priority的动态广播和静态广播，动态广播会排在前面**。

2>先接收的BroadcastReceiver可以对此有序广播进行截断，使后面的BroadcastReceiver不再接收到此广播，也可以对广播进行修改，使后面的BroadcastReceiver接收到广播后解析得到错误的参数值。当然，一般情况下，不建议对有序广播进行此类操作，尤其是针对系统中的有序广播

接收广播按顺序接收
**先接收的广播接收者可以对广播进行截断，即后接收的广播接收者不再接收到此广播；**
**先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播**
**具体使用**
有序广播的使用过程与普通广播非常类似，差异仅在于广播的发送方式：

```java
sendOrderedBroadcast(intent);
```



###### 6.4.2	App应用内广播(Local  Broadcast)

- 背景

  Android中的广播可以跨App直接通信(exported对于有intent-filter情况下默认值为true)

- 冲突

  可能出现的问题：

  - 其他App针对性发出与当前App   Intent-filter相匹配的广播，由此导致当前的APP不断接收广播并处理

  - 其他App注册与当前的App一致的intent-filter用于接收广播，并获取广播的具体信息

    

- 解决方案

  使用APP 应用内广播(Local  Broadcast)

  >1，APP应用内广播可理解为一种局部广播，广播的发送者和接受者都属于一个App
>
  >2，相比于全局广播(普通广播)，App应用内广播优势体现在：安全性&效率高

- 具体实现——将全局广播设置成全局广播

  - 注册广播时将exported书香设置为false，使得非本App内部发出的此广播不被接收

  - 在广播发送和接受时，增设相应权限permission，用于权限验证

  - 发送广播时指定该广播接收器所在的包名，此广播指挥发送到磁暴中的App内阈值相匹配的邮箱广播接收器中

    >通过intent.setPackage(packageName)指定包名

具体使用2 - 使用封装好的LocalBroadcastManager类
使用方式上与全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例

> 注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册

```java
//注册应用内广播接收器
//步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
mBroadcastReceiver = new mBroadcastReceiver(); 
IntentFilter intentFilter = new IntentFilter(); 

//步骤2：实例化LocalBroadcastManager的实例
localBroadcastManager = LocalBroadcastManager.getInstance(this);

//步骤3：设置接收广播的类型 
intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

//步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);

//取消注册应用内广播接收器
localBroadcastManager.unregisterReceiver(mBroadcastReceiver);

//发送应用内广播
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
localBroadcastManager.sendBroadcast(intent);
```

Android中的广播可以跨进程甚至跨App直接通信，且注册是exported对于有intent-filter的情况下默认值是true，由此将可能出现安全隐患如下：

1.其他App可能会针对性的发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收到广播并处理；

2.其他App可以注册与当前App一致的intent-filter用于接收广播，获取广播具体信息。

无论哪种情形，这些安全隐患都确实是存在的。由此，最常见的增加安全性的方案是：

1.对于同一App内部发送和接收广播，将exported属性人为设置成false，使得非本App内部发出的此广播不被接收；

2.在广播发送和接收时，都增加上相应的permission，用于权限验证；

3.发送广播时，指定特定广播接收器所在的包名，具体是通过intent.setPackage(packageName)指定在，这样此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

App应用内广播可以理解成一种局部广播的形式，广播的发送者和接收者都同属于一个App。实际的业务需求中，App应用内广播确实可能需要用到。同时，之所以使用应用内广播时，而不是使用全局广播的形式，更多的考虑到的是Android广播机制中的安全性问题。

相比于全局广播，App应用内广播优势体现在：

1.安全性更高；

2.更加高效。

为此，Android v4兼容包中给出了封装好的LocalBroadcastManager类，用于统一处理App应用内的广播问题，使用方式上与通常的全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将主调context变成了LocalBroadcastManager的单一实例。

代码片段如下：

```java
  //registerReceiver(mBroadcastReceiver, intentFilter);
  //注册应用内广播接收器
  localBroadcastManager = LocalBroadcastManager.getInstance(this);
  localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
          
  //unregisterReceiver(mBroadcastReceiver);
  //取消注册应用内广播接收器
  localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
  
 Intent intent = new Intent();
 intent.setAction(BROADCAST_ACTION);
 intent.putExtra("name", "qqyumidi");
 //sendBroadcast(intent);
 //发送应用内广播
 localBroadcastManager.sendBroadcast(intent);
```



**4.不同注册方式的广播接收器回调onReceive(context, intent)中的context具体类型**

1).对于静态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是ReceiverRestrictedContext；

2).对于全局广播的动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Activity Context；

3).对于通过LocalBroadcastManager动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Application Context。

注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册的ContextReceiver才有可能接收到（静态注册或其他方式动态注册的ContextReceiver是接收不到的）。

###### 5，粘性广播（Sticky Broadcast）

由于在Android5.0 & API 21中已经失效，所以不建议使用，在这里也不作过多的总结。

## 七，总结

对于不同的注册方式的广播接收器回调onReceive(Context context，Intent intent)中的context返回值时不一样的：

- 对于静态注册(全局+应用内广播)，回调onReceive(context, intent)中的context返回值是：ReceiverRestrictedContext；
- 对于全局广播的动态注册，回调onReceive(context, intent)中的context返回值是：Activity Context；
- 对于应用内广播的动态注册（LocalBroadcastManager方式），回调onReceive(context, intent)中的context返回值是：Application Context。
- 对于应用内广播的动态注册（非LocalBroadcastManager方式），回调onReceive(context, intent)中的context返回值是：Activity Context；
  