---
layout: post
title:  "[Android]Android_Service详解"
date:   2015-06-14 20:00:10
categories: 17---【Android】#########
comments: true
---

### 一、Service简介：
Service组件是与Activity组件最相似的组件。Service组件也是可执行的程序，有自己的生命周期。

![图片](http://owk5gjdrg.bkt.clouddn.com/0026Android_Service%E8%AF%A6%E8%A7%A3.png)

开发Service的步骤：

1、定义一个继承Service的子类。

2、配置Service。

![图片](http://owk5gjdrg.bkt.clouddn.com/0027Android_Service%E8%AF%A6%E8%A7%A3.png)

### 二、Service的生命周期：

context.startService() ->onCreate()- >onStart()->Service running--调用context.stopService() ->onDestroy() 

context.bindService()->onCreate()->onBind()->Service running--调用>onUnbind() -> onDestroy() 

下面通过一个实例进行说明：
```java

public class MyService extends Service {  
    private final String TAG = "MyService";  
  
    @Override  
    public void onCreate() {  
        // TODO Auto-generated method stub  
        super.onCreate();  
        Log.i(TAG, "--->>onCreate");  
    }  
  
    //onStartCommand方法中链接网络获取数据，关闭service服务  
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        // TODO Auto-generated method stub  
        Log.i(TAG, "--->>onStartCommand-->>"+intent.getStringExtra("name"));  
        return super.onStartCommand(intent, flags, startId);  
    }  
  
    @Override  
    public IBinder onBind(Intent arg0) {  
        // TODO Auto-generated method stub  
        return null;  
    }  
    //资源的释放  
    @Override  
    public void onDestroy() {  
        // TODO Auto-generated method stub  
        Log.i(TAG, "--->>onDestroy");  
        super.onDestroy();  
    }  
}  
```
```java
public class MainActivity extends Activity {  
    private Button start;  
    private Button stop;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        start = (Button) this.findViewById(R.id.button1);  
        stop = (Button) this.findViewById(R.id.button2);  
        start.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View arg0) {  
                // TODO Auto-generated method stub  
                Intent intent = new Intent(MainActivity.this, MyService.class);  
                intent.putExtra("name", "jack");  
                startService(intent);  
            }  
        });  
        stop.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                // TODO Auto-generated method stub  
                Intent intent = new Intent(MainActivity.this, MyService.class);  
                stopService(intent);  
            }  
        });  
    }  
  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
}  

```

### 三、绑定Service并实现通信：
如果Service和访问者之间需要进行方法调用和数据交换，则应该使用bindService()和unbindService()方法启动、关闭Service。

Context的bindService(Intent service,ServiceConnection conn,int flags)方法:

service:指定要启动的Service。

conn：该参数是一个ServiceConnection对象，用于监听访问者与Service之间的连接状况。

flags：指定绑定时是否自动创建Service。

下面有一个实例进行说明：
```java
public class BindService extends Service  
{  
    private int count;  
    private boolean quit;  
    // 定义onBinder方法所返回的对象  
    private MyBinder binder = new MyBinder();  
    // 通过继承Binder来实现IBinder类  
    public class MyBinder extends Binder //①  
    {  
        public int getCount()  
        {  
            // 获取Service的运行状态：count  
            return count;  
        }  
    }  
    // 必须实现的方法，绑定该Service时回调该方法  
    @Override  
    public IBinder onBind(Intent intent)  
    {  
        System.out.println("Service is Binded");  
        // 返回IBinder对象  
        return binder;  
    }  
    // Service被创建时回调该方法。  
    @Override  
    public void onCreate()  
    {  
        super.onCreate();  
        System.out.println("Service is Created");  
        // 启动一条线程、动态地修改count状态值  
        new Thread()  
        {  
            @Override  
            public void run()  
            {  
                while (!quit)  
                {  
                    try  
                    {  
                        Thread.sleep(1000);  
                    }  
                    catch (InterruptedException e)  
                    {  
                    }  
                    count++;  
                }  
            }  
        }.start();  
    }  
    // Service被断开连接时回调该方法  
    @Override  
    public boolean onUnbind(Intent intent)  
    {  
        System.out.println("Service is Unbinded");  
        return true;  
    }  
    // Service被关闭之前回调该方法。  
    @Override  
    public void onDestroy()  
    {  
        super.onDestroy();  
        this.quit = true;  
        System.out.println("Service is Destroyed");  
    }  
}  
```
```java
public class BindServiceTest extends Activity  
{  
    Button bind, unbind, getServiceStatus;  
    // 保持所启动的Service的IBinder对象  
    BindService.MyBinder binder;  
    // 定义一个ServiceConnection对象  
    private ServiceConnection conn = new ServiceConnection()  
    {  
        // 当该Activity与Service连接成功时回调该方法  
        @Override  
        public void onServiceConnected(ComponentName name  
            , IBinder service)  
        {  
            System.out.println("--Service Connected--");  
            // 获取Service的onBind方法所返回的MyBinder对象  
            binder = (BindService.MyBinder) service; //①  
        }  
  
        // 当该Activity与Service断开连接时回调该方法  
        @Override  
        public void onServiceDisconnected(ComponentName name)  
        {  
            System.out.println("--Service Disconnected--");  
        }  
    };  
  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        // 获取程序界面中的start、stop、getServiceStatus按钮  
        bind = (Button) findViewById(R.id.bind);  
        unbind = (Button) findViewById(R.id.unbind);  
        getServiceStatus = (Button) findViewById(R.id.getServiceStatus);  
        // 创建启动Service的Intent  
        final Intent intent = new Intent();  
        // 为Intent设置Action属性  
        intent.setAction("org.crazyit.service.BIND_SERVICE");  
        bind.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View source)  
            {  
                // 绑定指定Serivce  
                bindService(intent, conn, Service.BIND_AUTO_CREATE);  
            }  
        });  
        unbind.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View source)  
            {  
                // 解除绑定Serivce  
                unbindService(conn);  
            }  
        });  
        getServiceStatus.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View source)  
            {  
                // 获取、并显示Service的count值  
                Toast.makeText(BindServiceTest.this,  
                    "Serivce的count值为：" + binder.getCount(),  
                    Toast.LENGTH_SHORT).show(); //②  
            }  
        });  
    }  
}  
```
### 四、IntentService的使用：
Service本身不会创建一个线程，因此Service不能处理耗时任务。

IntentService刚好弥补这一缺陷。
```java
public class MyIntentService extends IntentService {  
  
    public MyIntentService() {  
        super("MyIntentService");  
    }  
  
    @Override  
    protected void onHandleIntent(Intent intent) {  
        Log.d("MyIntentService", "Thread id is " + Thread.currentThread().getId());  
    }  
  
    @Override  
    public void onDestroy() {  
        super.onDestroy();  
        Log.d("MyIntentService", "onDestroy executed");  
    }  
  
}  

```
```java
public class MainActivity extends Activity implements OnClickListener {  
  
    private Button startIntentService;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        startIntentService = (Button) findViewById(R.id.start_intent_service);  
        startIntentService.setOnClickListener(this);  
    }  
  
    @Override  
    public void onClick(View v) {  
        switch (v.getId()) {  
        case R.id.start_intent_service:  
            Log.d("MainActivity", "Thread id is " + Thread.currentThread().getId());  
            Intent intentService = new Intent(this, MyIntentService.class);  
            startService(intentService);  
            break;  
        default:  
            break;  
        }  
    }  
}  

```
### 五、跨进程调用Service(AIDL Service)
Android需要AIDL来定义远程接口。

AIDL的语法与java接口相似，但存在以下两点差异：

1、AIDL定义接口的源代码必须以.aidl结尾。

2、AIDL接口用到的数据类型除了基本类型外，其他类型全需要导包。

ICat.aidl
```java
package org.husheng.service;  
  
interface ICat  
{  
    String getColor();  
    double getWeight();  
}  

```
AidlService.java
```java
public class AidlService extends Service  
{  
    private CatBinder catBinder;  
    Timer timer = new Timer();  
    String[] colors = new String[]{  
        "红色",  
        "黄色",  
        "黑色"  
    };  
    double[] weights = new double[]{  
        2.3,  
        3.1,  
        1.58  
    };  
    private String color;  
    private double weight;  
    // 继承Stub，也就是实现额ICat接口，并实现了IBinder接口  
    public class CatBinder extends Stub  
    {  
        @Override  
        public String getColor() throws RemoteException  
        {  
            return color;  
        }  
        @Override  
        public double getWeight() throws RemoteException  
        {  
            return weight;  
        }  
    }  
    @Override  
    public void onCreate()  
    {  
        super.onCreate();  
        catBinder = new CatBinder();  
        timer.schedule(new TimerTask()  
        {  
            @Override  
            public void run()  
            {  
                // 随机地改变Service组件内color、weight属性的值。  
                int rand = (int)(Math.random() * 3);  
                color = colors[rand];  
                weight = weights[rand];  
                System.out.println("--------" + rand);  
            }  
        } , 0 , 800);  
    }  
    @Override  
    public IBinder onBind(Intent arg0)  
    {  
        /* 返回catBinder对象 
         * 在绑定本地Service的情况下，该catBinder对象会直接 
         * 传给客户端的ServiceConnection对象 
         * 的onServiceConnected方法的第二个参数； 
         * 在绑定远程Service的情况下，只将catBinder对象的代理 
         * 传给客户端的ServiceConnection对象 
         * 的onServiceConnected方法的第二个参数； 
         */  
        return catBinder; //①  
    }  
    @Override  
    public void onDestroy()  
    {  
        timer.cancel();  
    }  
}  
```
AidlClient.java
```java
public class AidlClient extends Activity  
{  
    private ICat catService;  
    private Button get;  
    EditText color, weight;  
    private ServiceConnection conn = new ServiceConnection()  
    {  
        @Override  
        public void onServiceConnected(ComponentName name  
            , IBinder service)  
        {  
            // 获取远程Service的onBind方法返回的对象的代理  
            catService = ICat.Stub.asInterface(service);  
        }  
  
        @Override  
        public void onServiceDisconnected(ComponentName name)  
        {  
            catService = null;  
        }  
    };  
  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        get = (Button) findViewById(R.id.get);  
        color = (EditText) findViewById(R.id.color);  
        weight = (EditText) findViewById(R.id.weight);  
        // 创建所需绑定的Service的Intent  
        Intent intent = new Intent();  
        intent.setAction("org.crazyit.aidl.action.AIDL_SERVICE");  
        // 绑定远程Service  
        bindService(intent, conn, Service.BIND_AUTO_CREATE);  
        get.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View arg0)  
            {  
                try  
                {  
                    // 获取、并显示远程Service的状态  
                    color.setText(catService.getColor());  
                    weight.setText(catService.getWeight() + "");  
                }  
                catch (RemoteException e)  
                {  
                    e.printStackTrace();  
                }  
            }  
        });  
    }  
  
    @Override  
    public void onDestroy()  
    {  
        super.onDestroy();  
        // 解除绑定  
        this.unbindService(conn);  
    }  
}  
```

### 六、总结：
掌握Service的用法。