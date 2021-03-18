# Android中的常用权限分析

Android提供了辅助（Accessibility）功能和服务帮助这些用户更加简单的操作设备，辅助功能的实质是监听应用窗口的变化和事件。

~~~java
Intent accIntent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
startActivity(accIntent);
~~~

以上就可以打开相关页面，赋予权限。

## 设备权限管理

需申请一下权限

~~~xml
android:permission="android.app.BIND_DEVICE_ADMIN"
~~~

然后在代码里面实现界面跳转，然后让用户选择是否授权。

## 通知栏管理权限

申请之后用户的通知栏信息就会被接管。

## VPN开发权限

Android 4.0之后增加了VPN功能，一旦申请该权限，就代表这个设备的网络访问消息会被申请者接管。



