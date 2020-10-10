# Dalvik

## Dalvik 寄存器（32位）

其有两种不同的命名方法。

* P命名法：它会从第一参数开始命名，对于非静态方法function(int a,int b)中有一个局部变量，其命名规则如下：
  * 一共四个寄存器，p0=对象引用（this）,但是如果是静态方法的时候这里就会直接是第一个参数。
  * p1=参数a
  * p2=参数b
  * p3=局部变量1
* V命名法：它会从第一个局部变量开始进行命名，假如一个和上面相同的函数。
  * v0 = 变量1
  * v1=对象引用
  * V2 = 参数1
  * V3 = 参数2

## 数据类型

•  V void       只能用于返回值类型

•  Z boolean    变量值只能是 真(true) 或 假(false)

•  B byte      字节型（无符号）

•  S short     短整型

•  C char     字符型（有符号）

•  I int       整数型

•  J long（64位） 长整型 对于64位数据，用两个寄存器来存放

•  F float     浮点型 

•  D double(64位) 双精度浮点型

•  L java类类型 package.name.ObjectName Lpackage/name/ObjectName;

•  [  数组类型 [I表示一个整型一维数组，等于java中的int[]。注意多位数数组的维数最大为255个。[[I=int[][] [[[I=int[][][]

## 类，方法，字段表示方法

•类：

•Lpackage/name/ObjectName

•方法：

•Lpackage/name/ObjectName;->MethodName(参数签名)返回类型

•字段：

•Lpackage/name/ObjectName;->FieldName:类型

> Package = 包名
>
> ObjectName = 类名
>
> 参数签名：这个就是数据类型的集合，如果有两个int 型的参数，那么就是II

## 跳转指令

​    if-eq  等于则跳转 ==        equal to

•  if-ne  不等于则跳转 !=        not equal to

•  if-lt  小于则跳转 <         less than

•  if-ge  大于或等于则跳转 >=     greater than or equal to

•  if-gt  大于则跳转 >        greater than

•  if-le  小于或等于则跳转 <=     less than or equa to

•  if-eqz 等于0                 equal to zero

•  if-nez 不等于0               not equal to zero

•  goto  无条件跳转            

•  switch 分支跳转

## 举个栗子

~~~
已知v0指向一个StringBuffer对象
invoke-virtual {v0, v1}, Ljava/lang/StringBuffer;->append(I)Ljava/lang/StringBuffer;

const-string v0, "jni"  
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
~~~

invoke-virtual {v0, v1}这里是参数列表, Ljava/lang/StringBuffer;->append(I)这里是方法名和参数类型Ljava/lang/StringBuffer这里是返回值。;

