# Android开发与逆向常用命令总结

## 非shell命令

>  adb shell dumpsys activity top

* 查看应用当前的activity信息

>  adb shell dumpsys activity   这个会打印当前系统中所有应用运行的四大组件都会打印出来。

~~~bash
adb shell dumpsys package [packageName]//查看指定包名应用的详细信息
~~~

adb shell dumpsys meminfo  [pname/pid]//查看指定进程名或者进程id的内存信息

* 利用这个命令可以查看进程当前的内存情况，和top命令可以结合使用，分析应用的性能消耗情况。

abd shell dumpsys dbinfo   [packageName]

* 查看指定包名应用的数据库存储信息,甚至还可以看到应用执行过的sql语句信息。

adb install apk

adb uninstall [packageName]

adb pull 

* 将设备中的文件放到本地

  > adb pull /sdcard/tmp.txt D:\ 

adb push 

* 将本地文件放入设备中

  > adb push D:/tmp.txt /sdcard/tmp

adb shell screencap

* 截屏操作

  > adb shell screencap -p /sdcard/tmp.png

adb shell screenrecord

* 录屏操作

  > adb shell screenrecord /sdcard/tmp.mp4

adb shell input text [需要输入文本框的内容]

* 输入文本内容

  > adb shell input text 'HelloWorld'

  值得注意的是，这个命令也可以模拟物理按键，虚拟键盘，滑动，滚动等事件

adb forward 

* 端口转发

  > abd forward [（远程端）协议：端口号] [（设备端）协议:端口号]

adb jdwp

* 查看设备中可以被调式的应用的进程号

adb logcat 

* 查看当前的日志信息

  > adb logcat -s tag
  >
  > adb logcat | findstr pname/pid/keyword         //Windows条件下findstr信息过滤 linux 用grep

### shell命令

adb shell pm clear [pkgname] 

* 清空应用数据

run as 

* 可以在非root设备中查看指定debug模式的包名应用沙盒数据。

  > run-as [package name]

ps 

* 查看设备的进程信息，或者指定进程的线程的信息。

  > ps | grep 
  >
  > ps -t [pid]  //查看pid对应的线程的信息

pm install [apk文件] //安装设备中的apk

pm uninstall [packageName] //卸载

am start

* 启动一个应用

  > am start -n [包名]/[包名].[activity名称]
  >
  > am start -n com.android.browser/com.android.browser.BrowserActivity

  可以以debug的方式启动应用(am start -D -n ....)

am start service

* 启动一个服务

  > am startservice -n [包名]/[包名].[服务名]
  >
  > am startservice -n com.android.traffic/com.android.traffic.maniservice

am broadcast

* 发送一个广播

  > am broadcast -a [广播动作]
  >
  > am broadcast -a android.ENT.conn.CONNECTIVITY_CHANGE

netcfg (好像用不了了ifconfig可以)

* 查看设备的ip地址

netstat 

* 查看设备的端口号信息

app_process 

* 运行java代码

  > export CLASSPATH=/data/demo.jar
  >
  > exec /system/bin/app_process /data/.cn.wjdiankong.Main

dalvikvm 

* 运行一个dex文件

  > dalvikvm -cp [dex文件] [运行主类]
  >
  > dalvikvm -cp /data/demo.dex cn.wjdiankong.Main

top 

* 查看当前应用的CPU消耗信息

  > top [-n/-m/-d/-s/-l]
  >
  > -m 最多显示多少个进程
  >
  > -n 刷新次数
  >
  > -d 刷新时间间隔
  >
  > -s 按那列排序
  >
  > -t 显示线程信息而不是进程信息
  >
  > 各个设备上有所不同

getprop 

* 查看系统属性值

  > getprop ro.debuggable

  root设备之后这个命令还可以修改相关信息

## 操作apk命令

aapt dump xmltree [apk包] [需要查看的资源文件xml]

* aapt dump xmltree dmeo.apk AndroidManifest.xml

dexdump [文件路径]

## 进程命令

> 跟linux一样

cat /proc/[pid]/maps

cat /proc/[pid]/status //查看进程状态



cat /proc/[pid]/net/tcp/tcp6      /udp/udp6

* 获取当前应用使用的端口号信息
* 上面其实是4条命令





> 我发下第三章，第四章都是linux基础内容，直接tmd跳过

























