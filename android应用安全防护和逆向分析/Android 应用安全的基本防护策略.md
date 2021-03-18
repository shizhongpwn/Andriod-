# Android 应用安全的基本防护策略

## 混淆机制

`代码混淆`

反编译之后的apk，看到的代码类名，方法名及其代码格式看起来不像正常的Android项目代码。

破解查看Java层代码的方式：

* 直接先解压classes.dex文件，然后用dex2jar工具转化成jar文件，然后用jd-gui工具查看。
* 使用apktool工具直接反编译apk，得到smali源码，阅读smali源码
* 都不如我jeb来的实在

`资源混淆`

微信团队开源混淆代码：

> https://github.com/shwenzhang/AndResGuard

但是他是对资源名的混淆，并没有混淆资源内容，同时我们直接查看java代码的时候是直接看到的资源id，那么我们可以通过资源id直接去res/values文件夹下找到对应的资源内容

## 签名保护

Android中的每一个应用都有一个唯一的签名，一个应用没有签名是不被允许安装到设备中的，在以往的破解中都是反编译之后重新打包。

> 签名在反编译之后是无法得到的

如何防止被重新打包？

> 在程序入口的地方添加签名验证，如果发现应用的签名不正确，就立即退出程序。
>
> 但是上述方法存在问题，如果我们在java层做签名认证，其实安全性是非常低的，破解者可以通过反编译修改smali代码，使得签名验证功能失效。
>
> 或者可以考虑签名方法做在native层，不过还是可能被找到修改

## 手动注册native方法

so加载流程：

1. java层运行System.loadLibrary("jnitest");
2. 程序载入相关库并产生Load事件，事件触发之后，程序默认会在载入的.so文件的函数列表中搜索JNI_OnLoad函数并执行。
3. 当.so文件被卸载的时候，同样会触发Unload事件，程序默认会去在载入的.so文件的函数列表中查找JNI_OnUnload函数并执行，然后卸载.so文件。

一般情况下，在C组件中的JNI_OnLoad函数用来实现给VM注册接口，以便VM可以快速地找到Java代码需要调用的C函数。（JNI_OnLoad还可以告诉VM此C组件使用的JNi版本）。

但是如果每次调用VM都要到so里面搜索函数，效率低下，所以可以再VM内部注册native函数，通过在JNI_OnLoad函数中先行注册native函数到VM的native链表上去，进而加快了调用速度。

**native层函数对应的函数名一般为Java+类名+方法名**

缺点：

* 容易被ida找到
* 攻击者得到这个so之后，查看这个native方法的参数和返回值也就是方法签名，然后再java层自己写个demo，然后构造一个和so文件中对应的native方法，就可以执行这个native方法，获取到执行结果。

通过显示的注册jni方法，可以有所缓解，只需要在native层代码中调用如下三个函数即可：

* (*env)->RegisterNatives(env, clazz, methods, methodsLenght)
  * ps其实之前的基础里面有
  * clazz肯定是jclass类
  * methods是结构体,`JNINativehod`

~~~java
public native int  obviousEqual();
public native void checkJSK();
~~~

~~~java
#include <assert.h>
#define JNIREG_CLASS "com/code/jsk/handlenative/MainActivity"//指定要注册的类
static JNINativeMethod gmethod[] = {
        {"obviousEqual","()I",(void*)jiangsikang},
};
/*
* Register several native methods for one class.
*/
static int registerNativeMethods(JNIEnv* env, const char* className,
                                 JNINativeMethod* gmethod, int numMethods)
{
    jclass clazz;
    clazz = env->FindClass(className);
    if (clazz == NULL) {
        return JNI_FALSE;
    }
    if (env->RegisterNatives(clazz, gmethod, numMethods) < 0) {
        return JNI_FALSE;
    }

    return JNI_TRUE;
}
/*
* Register native methods for all classes we know about.
*/
static int registerNatives(JNIEnv* env)
{
    if (!registerNativeMethods(env,JNIREG_CLASS, gmethod,
                               sizeof(gmethod) / sizeof(gmethod[0])))
        return JNI_FALSE;
    JNINativeMethod smethod[] = {
            {"checkJSK","()V",(void*)checkJSK},
    };
    if (!registerNativeMethods(env,JNIREG_CLASS, smethod,
                               sizeof(smethod) / sizeof(smethod[0])))
        return JNI_FALSE;
    return JNI_TRUE;
}
/*
* Set some test stuff up.
*
* Returns the JNI version on success, -1 on failure.
*/

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        return -1;
    }
    assert(env != NULL);

    if (!registerNatives(env)) {//注册
        return -1;
    }

    result = JNI_VERSION_1_4;

    return result;
}
~~~

以上就实现了隐藏式的函数注册，其实mthod结构体是实现函数映射的关键，我们可以通过修改结构体C函数指针的名称，使得攻击者在so库里面无法直接通过字符串找到对应的java层函数。

破解者：

此时破解者无法直接找到对应的so库函数，那么必然去寻找JNI_OnLoad函数，然后通过分析ARM汇编代码找到register函数，分析注册结构体，找到对应的native方法，不过总归是骗过了小白破解者。

还有就是JNI_OnLoad函数可以用于签名验证，增加安全性，native层的代码分析难度稍微大一点，同时这种签名机制必须静态破解，因为程序在签名保护下是跑不起来的。

## 反调试检测

反调试检测是为了应对破解者的ida动态调试so文件，IDA动态调试是基于进程的注入技术，然后使用Linux中的ptrace机制，进行调试目标进程的附加操作，**ptrace机制有一个特点，如果一个进程被调试了，那么在它进程的status文件中有一个字段TracerPid会记录调试者的进程id值。**

那么反调试的一个方法就很明显了，进程启动前先检查自己的这个文件，然后拿到这个值，如果大于0，直接退出进程。

破解者：

​	这个时候自然要在反调试代码被加载前断下进程，`JNI_OnLoad`函数是一个不二的选择，然后找到检测轮询代码，使用nop指令，替换检测指令。

























