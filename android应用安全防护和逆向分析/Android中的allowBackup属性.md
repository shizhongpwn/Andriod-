



# Android中的allowBackup属性

开发者在使用IDE工具开发的时候自动生成代码，但是这个属性会被设置为true，这会导致隐私相关数据丢失。

Android API Level 8及以上Android系统提供了为应用程序数据的备份和恢复功能，该功能由`allowBackup`属性决定，其属性默认值为true，此时用户可以通过`adb backup` 和 `adb restore` 命令对应用的数据进行备份和恢复。

abd backup 允许一个能够打开USB调试开关的人从Android手机中复制应用数据到外设，然后通过adb restore 可以指定该数据源把数据恢复到另一台设备上进行查看。

## 如何获取应用的隐私数据

其实步骤和上面的一样，通过备份命令可以得到.ab结尾的android的备份文件

可以使用`abe.jar`工具将改文件解析为.tar文件进而拿到数据。



