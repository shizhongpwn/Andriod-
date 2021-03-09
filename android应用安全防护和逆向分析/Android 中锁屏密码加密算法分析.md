# Android 中锁屏密码加密算法分析

## 密码算法分析

在Android中，如果发现获取服务的方式`ServiceManager.getService("lock_settings")`获取到lockSettingsService类。



这其实没啥意思的，手势密码其实把九宫格图案转化成数组然后呢SHA1加密之类的加密算法。