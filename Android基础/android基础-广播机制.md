# android基础-广播机制

> ip网络里面，最大的IP地址是保留作为广播地址来使用的。比如192.168.0.XXX , 子网掩码255.255.255.0，那么这个网络的广播地址就是192.168.0.255.

Android里面的广播类型：

* 标准广播：完全异步的方式执行广播，所有的广播接收器都可以接收这条广播，因此他们之间没有任何先后顺序可言，同时无法截断。

![image-20201020152719336](android基础-广播机制.assets/image-20201020152719336.png)

* 有序广播：同步执行广播，广播发出之后只有一个广播接收器能够接收这条广播信息，当前广播里面的执行逻辑全部结束之后才会执行下一个，优先级高的广播器还可以进行广播的截断，让其他广播接收器无法接收拿到广播信息。

![image-20201020152939732](android基础-广播机制.assets/image-20201020152939732.png)

## 动态注册监听网络变化

> 注册广播的方式一般有两种：在代码里面注册和在androidManifest.xml里面进行注册，其中前者被称为动态注册，后者被称为静态注册。

~~~java
public class MainActivity extends AppCompatActivity {
    private IntentFilter intentFilter;
    private NetworkChangeReceicer networkChangeReceicer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intentFilter = new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceicer = new NetworkChangeReceicer();
        registerReceiver(networkChangeReceicer,intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(networkChangeReceicer);
    }

    class NetworkChangeReceicer extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context,"network changes",Toast.LENGTH_SHORT).show();
        }
    }
}
~~~

讲解：

* NetworkChangeReceicer类继承自BroadcastReceiver类，在类的内部，我们重写了父类的onReceive方法，这样，每次网络状态变化的时候，都会执行悬浮框代码。
* onCreate()方法，我们创建了一个IntentFilter实例，添加了一个"android.net.conn.CONNECTIVITY_CHANGE"的action（当网络状态发生变化的时候，系统发出的就是一个android.net.conn.CONNECTIVITY的广播，所以我们想监听什么信息就从这里添加相应的action）
* 创建一个NetworkChangeReceicer实例，然后调用registerReceiver()方法进行注册，注意它的参数，这样NetwordChangeReceiver就会收到所有值为android.net.conn.CONNECTIVITY_CHANGE的广播，这样就实现了监听网络变化的功能。
* 动态注册的广播接收器一定要取消注册，我们在onDestory()方法里面通过调用unregisterReceiver()方法取消注册。

~~~java
  class NetworkChangeReceicer extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
            if(networkInfo != null && networkInfo.isAvailable())
            {
                Toast.makeText(context,"network is available",Toast.LENGTH_SHORT).show();
            }
            else
            {
                Toast.makeText(context,"network is unavailable",Toast.LENGTH_SHORT).show();
            }
        }
    }
~~~

> 以上的操作android访问了用户的设备和隐私，这些敏感操作都必须在AndroidManifest.xml里面加入权限，比如上面的，我们就需要加上对网络系统的访问权限：

~~~xml
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
~~~

