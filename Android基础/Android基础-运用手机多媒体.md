# Android基础-运用手机多媒体

## 通知功能

>  书上的安卓版本较低，在高安卓版本上，有些许差异

我自己写的`demo`，适用于高版本：

~~~java
·package com.example.notificationtest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.NotificationCompat;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Build;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private static final String CHANNEL_ID = "test1";

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.button:
                createNotificationChannel();
                NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
                Notification notification = new NotificationCompat.Builder(this,CHANNEL_ID)
                        .setContentTitle("This is a Content title")
                        .setContentText("This is a Content text")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                        .setPriority(Notification.PRIORITY_DEFAULT)
                        .build();
                notificationManager.notify(1,notification);
                break;
            default:
                break;
        }
    }
    private void createNotificationChannel(){
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            CharSequence name = getString(R.string.channel_name1);
            String description = getString(R.string.channel_des);
            int importance = NotificationManager.IMPORTANCE_DEFAULT;
            NotificationChannel channel = new NotificationChannel(CHANNEL_ID,name,importance);
            channel.setDescription(description);
            NotificationManager notificationManager = getSystemService(NotificationManager.class);
            notificationManager.createNotificationChannel(channel);
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button);
        button.setOnClickListener(this);
    }
}
~~~

在低版本Android上是不需要`createNotificationChannel`函数来创建信道的，同时也不需要`setPriority()`函数来设置优先级的。

> https://developer.android.com/training/notify-user/build-notification?hl=zh-cn

必须先通过向 `createNotificationChannel()` 传递 `NotificationChannel` 的实例在系统中注册应用的[通知渠道](https://developer.android.com/training/notify-user/channels?hl=zh-cn)，然后才能在 Android 8.0 及更高版本上提供通知。因此以下代码会被 `SDK_INT` 版本上的条件阻止。

步骤：

1. 创建相应的`channal`
2. 通过`getSystemService()`,`NotificationManager`
3. 添加相应的操作。
4. 以`notify()`显示通知。

`PendingIntent`

其倾向于在某个合适的时机去执行某个动作。

通过静态方法获取`PendingIntent`实例

* getActivity()方法
* getBroadcast()方法
* getService() 方法

这几个方法参数相同。

1. Context参数
2. 第二个参数通常用不到，一般传入0
3. `Intent`对象
4. 用于确定`PendingIntent`行为。通常情况传0，特殊情况查参数含义。
   1. `FLAG_ONE_SHOT`
   2. `FLAG_NO_CREATE`
   3. `FLAG_CANCEL`
   4. `FLAG_ UPDATE_CURRENT`

`通知栏关闭`

上面的`demo`的不会被关闭，同时也无法引发相关操作，看下面的：

~~~java
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.button:
                createNotificationChannel();
                Intent intent = new Intent(this,Notification_Activity.class);
                PendingIntent pi = PendingIntent.getActivity(this,0,intent,0);
                NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
                Notification notification = new NotificationCompat.Builder(this,CHANNEL_ID)
                        .setContentTitle("This is a Content title")
                        .setContentText("This is a Content text")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                        .setPriority(Notification.PRIORITY_DEFAULT)
                        .setContentIntent(pi)
                        .setAutoCancel(true)
                        .build();
                notificationManager.notify(1,notification);
                break;
            default:
                break;
        }
    }
~~~

关键区别在于：`setContentIntent`方法和`setAutoCancel()`方法，后者使得我们在点击通知栏之后，通知栏可以消失，这一步也可以用下面代替：

~~~java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button);
        button.setOnClickListener(this);
        NotificationManager notificationManager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
        notificationManager.cancel(1);
    }
~~~

通过`NotificationManager`自带的cancel方法，通过相关`ID`来关闭通知栏。

`通知的进阶技巧`

说一些API吧

~~~
.setSound() //通知声音
.setVibreate() //通知震动,需要 VIBRATE 权限
.setLights() // 通知LED 闪烁
.setDefaults() // 默认设置
~~~

`通知的高级功能`

~~~java
.setStyle(new NotificationCompat.BigTextStyle().bigText("good"))//方法可以构造出丰富文本的通知内容
//也可以用来显示一个大图片
.setStyle(new NotificationCompat.BigPictureSyle().bigPicture(BitmapFactory.decodeResource(getResources(),R.drawable.big_image))).build();
~~~

~~~java
.setPriority() //用于设置通知的重要程度
~~~

参数如下：

* `PRIORITY_DEFAULT` 默认
* `PRIORITY_MIN` 最低重要程度，只有在特定场景下才会显示这个通知。
* `PRIORITY_LOW` 较低的通知程度，系统可能会将此类通知缩小，或者改变其显示的顺序。
* `PRIORITY_HIGH` 较高的通知程度，系统可能会将此类通知放大，或者将其放在前面的位置。
* `PRIORITY_MAX` 较高的通知程序，必须立刻让用户看到。

~~~java
.setPriority(NotificationCompat.PRIORITY_MAX);
~~~

`调用摄像头`

~~~java
package com.example.cameraalbumtest;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;

import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.media.Image;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.MediaStore;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;

import java.io.File;
import java.io.IOException;

public class MainActivity extends AppCompatActivity {
    public static final int TAKE_PHOTO = 1;
    private ImageView picture;
    private Uri imageUri;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button takePhoto = (Button)findViewById(R.id.button2);
        picture = (ImageView)findViewById(R.id.imageView);
        takePhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                File outputImage = new File(getExternalCacheDir(),"output_image.jpg");
                //创建FILE对象，用于存储照片。
              
                try {
                    if(outputImage.exists()){
                        outputImage.delete();
                    }
                    outputImage.createNewFile();
                }catch (IOException e){
                    e.printStackTrace();
                }
                if(Build.VERSION.SDK_INT>=24){
                    imageUri = FileProvider.getUriForFile(MainActivity.this,"com.example.cameraalbumtest.fileprovider",outputImage);
                }
                else {
                    imageUri = Uri.fromFile(outputImage);
                }
                //启动相机
                Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
                intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri); //隐式的Intent，系统会找出能够响应这个Intent的活动去启动
                startActivityForResult(intent,TAKE_PHOTO);
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    try {
                        Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                        picture.setImageBitmap(bitmap);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                break;
            default:
                break;
        }
    }
}
~~~

* `getExternalCacheDir()`方法可以得到手机SD卡的应用关联目录（就是指SD卡中专门用于存放当前应用缓存数据的位置。）
  * 其具体路径`/sdcard/Android/data/<package name>/cache`
  * 为什么用`应用关联目录`？
    * 从`Andorid6.0`开始，读写`SD`卡被视为危险权限，如果图片放在SD卡的任何其他目录，都要进行运行时的权限处理，用关联目录可以跳过这一步。
* 系统版本低于`Android7.0`的话，就直接用`Uri`的`fromFile()`方法进行对象转换。
  * 高于的话，`getUriForFile()`方法将File对象转换成一个封装过的`Uri`对象。
  * `Android7.0`开始，直接用本地真实路径被认为不安全，`FileProvider`是一种特殊的内容提供器，对数据进行保护，选择性的将封装过的`Uri`共享给外部。
* `Intent`的`putExtra()`方法用来指定图片的输出地址。



















































































