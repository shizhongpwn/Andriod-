# Android特色开发-基于位置的服务

## 简介

GPS定位的工作原理是基于手机内置的GPS硬件直接合卫星交互来获得当前 经纬度的信息，其优点在于精度高，但是只能在室外使用。

网络定位的工作原理是根据手机当前网络附近的三个基站进行测速，以此计算出手机和每个基站之间的位置，然后通过三角定位来得到一个大概的位置。缺点在于精度差，优点在于室内室外都可使用。

还有一种是wi-fi定位。

第三方SDK可以帮我们解决上面的问题，国内百度和高德做的不错。

## 申请API Key

申请成功百度开发者

http://developer.baidu.com/user/reg

GQXWu30G0qTRH4wL1ENhYSf9

## 使用百度定位（Android 导入 第三方SDK）

这一点偏为麻烦的还是jar包的导入，首先在

![image-20210221091250969](Android特色开发-基于位置的服务.assets/image-20210221091250969.png)

这里点+，然后选择导入jar包，然后写入路径，然后还需要把依赖的so文件放到合适的位置。

![image-20210221091619270](Android特色开发-基于位置的服务.assets/image-20210221091619270.png)

jniLibs文件夹是自己建立在main文件夹下面的。

~~~java
package com.example.lbstest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.TextView;
import android.widget.Toast;

import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.LocationClient;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    public LocationClient locationClient;
    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //getApplicationContext()可以获得一个全局的Context
        locationClient = new LocationClient(getApplicationContext());//创建一个百度jar包里面的LocationClient实例，参数为一个Context参数
        locationClient.registerLocationListener(new MyLocationListener());//注册一个定位监听器，当获取到位置信息的时候就会回调这个定位监听器
        textView = (TextView)findViewById(R.id.position_text_view);
        /*
        利用List集合收集没有被授予的权限，注意因为ACCESS_FINE_LOCATION和ACCESS_COARSE_LOCATION同属于一个权限组，所以只用申请他们中的一个就可以了
        最后转换为一个数组，然后进行权限请求
        * */
        List<String> permissionList = new ArrayList<>();
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.ACCESS_FINE_LOCATION)!= PackageManager.PERMISSION_GRANTED)
        {
            permissionList.add(Manifest.permission.ACCESS_FINE_LOCATION);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.READ_PHONE_STATE)!=PackageManager.PERMISSION_GRANTED)
        {
            permissionList.add(Manifest.permission.READ_PHONE_STATE);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED)
        {
            permissionList.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
        }
        if(!permissionList.isEmpty()){
            String []permissions = permissionList.toArray(new String[permissionList.size()]); //转化为数组
            ActivityCompat.requestPermissions(MainActivity.this,permissions,1);//权限请求
        }else {
            requestLocation();
        }
    }
    private void requestLocation(){
        locationClient.start();
    } //定位开始，定位的结果会回调到我们前面注册的监听器里面去。

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode){
            case 1:
                if(grantResults.length>0){ //grantResults代表请求结果
                    for (int result : grantResults){
                        if(result != PackageManager.PERMISSION_GRANTED){ //如果程序没有被赋予权限就关闭该程序，这里就表明，用户必须全部同意我们申请的权限才可以。
                            Toast.makeText(this,"必须同意权限才可使用本程序",Toast.LENGTH_LONG).show();
                            finish();
                        }
                    }
                }
                break;
            default:
                break;
        }
    }
    public class MyLocationListener implements BDLocationListener{
        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
            StringBuilder currentPosition = new StringBuilder();
            currentPosition.append("纬度：").append(bdLocation.getLatitude()).append("\n");//这里就可以直接利用jar包获得定位信息了
            currentPosition.append("经线：").append(bdLocation.getLongitude()).append("\n");
            currentPosition.append("定位方式：");
            if(bdLocation.getLocType()== BDLocation.TypeGpsLocation){//判断定位类型GPS，还是网络定位
                currentPosition.append("GPS");
            }else if (bdLocation.getLocType()== BDLocation.TypeNetWorkLocation){
                currentPosition.append("网络");
            }else if(bdLocation.getLocType()== BDLocation.TypeServerError) //模拟器一般都返回这个,tmd百度SDK api文档也没查到什么有用的信息，服务关了还能获得定位信息
            {
                currentPosition.append("服务未开启");
            }
            textView.setText(currentPosition);
        }
    }
}
~~~

到此为止，就完成了最基础的定位服务。

但是上面值进行了一次的定位，当位置移动的时候无法获得持续的定位结果，为了实现持续定位，只需要修改一个方法，增加两个方法：

~~~java
    private void requestLocation(){
        initLocation();
        locationClient.start();
    }
    private void initLocation(){
        LocationClientOption option = new LocationClientOption();//听名字就知道是设置LocationClient的属性的
        option.setScanSpan(5000); //定位时间间隔设置为5000毫秒，表示5秒进行一次定位
        locationClient.setLocOption(option);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        locationClient.stop(); //因为定义了重复定位，所以在程序销毁的时候要记得关闭定位功能，否则程序会在后台不断的持续定位。
    }
~~~

LBS SDK一共只有三种模式可以选择：

* Hight_Accuracy  高精度
* Battery_Saving  节电模式
* Device_Sensors  传感器模式，只会使用GPS进行定位

可以通过LocationClientOption来设置模式：

~~~java
    private void initLocation(){
        LocationClientOption option = new LocationClientOption();//听名字就知道是设置LocationClient的属性的
        option.setScanSpan(5000); //定位时间间隔设置为5000毫秒，表示5秒进行一次定位
        option.setLocationMode(LocationClientOption.LocationMode.Hight_Accuracy);
        locationClient.setLocOption(option);
    }

~~~

`如何看懂位置信息`

~~~java
    private void initLocation(){
        LocationClientOption option = new LocationClientOption();//听名字就知道是设置LocationClient的属性的
        option.setScanSpan(5000); //定位时间间隔设置为5000毫秒，表示5秒进行一次定位
        //option.setLocationMode(LocationClientOption.LocationMode.Hight_Accuracy);
        option.setIsNeedAddress(true);//设置打开详细的地址信息
        locationClient.setLocOption(option);
    }

~~~

~~~java
 public class MyLocationListener implements BDLocationListener{
        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
            StringBuilder currentPosition = new StringBuilder();
            currentPosition.append("纬度：").append(bdLocation.getLatitude()).append("\n");//这里就可以直接利用jar包获得定位信息了
            currentPosition.append("经线：").append(bdLocation.getLongitude()).append("\n");
            currentPosition.append("国家：").append(bdLocation.getCountry());
            currentPosition.append("省：").append(bdLocation.getProvince());
            currentPosition.append("城市：").append(bdLocation.getCity()).append("\n");
            currentPosition.append("区：").append(bdLocation.getDistrict()).append("\n");
            currentPosition.append("街道：").append(bdLocation.getStreet()).append("\n");
            currentPosition.append("定位方式：");
            if(bdLocation.getLocType()== BDLocation.TypeGpsLocation){//判断定位类型GPS，还是网络定位
                currentPosition.append("GPS");
            }else if (bdLocation.getLocType()== BDLocation.TypeNetWorkLocation){
                currentPosition.append("网络");
            }else if(bdLocation.getLocType()== BDLocation.TypeServerError) //模拟器一般都返回这个,tmd百度SDK api文档也没查到什么有用的信息，服务关了还能获得定位信息
            {
                currentPosition.append("服务未开启");
            }
            textView.setText(currentPosition);
        }
    }
~~~

## 使用百度地图

~~~xml
    <com.baidu.mapapi.map.MapView
        android:layout_height="match_parent"
        android:layout_width="match_parent"
        android:id="@+id/bmapView"
        android:clickable="true"
        />
~~~

~~~java
    private MapView mapView;
    ....
        protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SDKInitializer.initialize(getApplicationContext()); //SDK地图初始化，但是这个需要在serContentView之前进行调用
        setContentView(R.layout.activity_main);
        mapView = (MapView)findViewById(R.id.bmapView);
        ......
         @Override
    protected void onResume() {//onResume方法是Activity第一次创建时 重新加载实例时调用 例如 我打开App搜索第一个界面OnCreate完 就调用onResume 然后切换到下一个界面 第一个界面不finish 按Back键回来时 就调onResume 不调onCreate， 还有就是 App用到一半 有事Home键切出去了 在回来时调onResume ，在这里调用该方法的时候，恢复地图的运行
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {//当该activity被其他事件中断，或者被暂停的时候，通过该方法停止地图的调用。
        super.onPause();
        mapView.onPause();
    }
~~~

`地图移动到我的位置`

LBS SDK里面提供了一个BaiduMap，它是地图的总控制器

* getMap方法可以获得BaiduMap的实例

百度地图的缩放级别在3-19之间，小数点也可取，值越大，地图显示的信息越精细。

那么首先我们要获取一个地图的实例：

~~~java
    private BaiduMap baiduMap;
    private boolean isFirstLocate = true;
    ......
        @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SDKInitializer.initialize(getApplicationContext()); //SDK地图初始化，但是这个需要在serContentView之前进行调用
        setContentView(R.layout.activity_main);
        mapView = (MapView)findViewById(R.id.bmapView);
        baiduMap = mapView.getMap();//获取地图的实例
~~~

然后通过该实例实现一个位置转换方法

~~~java
    private void navigateTo(BDLocation location)
    {
        if(isFirstLocate){
            LatLng ll = new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update = MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);//将地图移动到指定的经纬度上面。
            update = MapStatusUpdateFactory.zoomTo(16f);//zoomTo方法接受一个float类型的参数用来设置缩放级别
            baiduMap.animateMapStatus(update);//更新缩放级别
            isFirstLocate = false;
        }
    }
~~~

LatLng类主要就是用来存放经纬度的值的，它的构造方法接受两个参数，第一个参数是纬度值，第二个参数是经度值，之后调用MapStatusUpdateFactory的newLatLng()方法将LatLng对象传入，newLatLng方法返回的也是一个MapStatusUpdate对象。

`让我显示在地图上`

百度LSB SDK提供了一个MyLocationData.Builder类，这个类用来封装设备当前的所在位置，只需要将经纬度信息传入到这个类的相应的方法里面就可以了。

传入之后利用build方法可以生成该类的实例。

修改之前移动地图位置的方法：

~~~java
    private void navigateTo(BDLocation location)//实现地图的位置移动
    {
        if(isFirstLocate){
            LatLng ll = new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update = MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);
            update = MapStatusUpdateFactory.zoomTo(16f);
            baiduMap.animateMapStatus(update);
            isFirstLocate = false;
        }
        MyLocationData.Builder locationData = new MyLocationData.Builder();
        locationData.latitude(location.getLatitude());
        locationData.longitude(location.getLongitude());
        MyLocationData myLocationData = locationData.build();
        baiduMap.setMyLocationData(myLocationData);
    }
~~~

可以看到我们构建了一个MyLocationData的实例，并将其设置到了BaiduMap里面，**到这里其实才明白isFirstLocate的存在其实是因为打开地图的时候把地图移动到机器这里只需要一次，但是因为之后的不断定位其实不用每次都移动到指定的经纬度上面**。

但是使用此功能我们必须使用BaiduMap的setMyLocationEnabled()方法打开此功能，这是由于百度地图SDK的限制。





















