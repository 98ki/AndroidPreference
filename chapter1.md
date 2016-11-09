# First Chapter

ANDROID内存面临的问题：

1.有限的堆内存，原始只有16M

2.内存大小消耗等根据设备，操作系统等级，屏幕尺寸的不同而不同

3.程序不能直接控制

4.支持后台多任务处理（multitasking）

5.运行在虚拟机之上

tips:

尽量避免static成员变量引用资源耗费过多的实例,比如Context

频繁操作字符串时，StringBuffer代替String。

对于所有所有基本类型的组合：int数组比Integer数组好

总体来说，就是避免创建短命的临时对象。减少对象的创建就能减少垃圾收集，进而减少对用户体验的影响。

避免使用浮点数

使用实体类比接口好，如果HashMap完全适合你的程序，那么使用Map就没有什么价值，速度会慢一倍。

避免使用枚举

了解并使用类库，很多类库是用c\/c++实现，如arrayCopy方法在有JIT的Nexus One上，自行编码的循环快9倍。String.indexOf\(\)，String.lastIndexOf\(\)等特殊实现的方法，比起Java循环快10到100倍。

android.text.format包下的Formatter类，提供了IP地址转换、文件大小转换等方法；DateFormat类，提供了各种时间转换，都是非常高效的方法。

TextUtils类，对于字符串处理Android为我们提供了一个简单实用的TextUtils类，如果处理比较简单的内容不用去思考正则表达式不妨试试这个在android.text.TextUtils的类

高性能MemoryFile类，对于I\/O需要频繁操作的，主要是和外部存储相关的I\/O操作，MemoryFile通过将 NAND或SD卡上的文件，分段映射到内存中进行修改处理，这样就用高速的RAM代替了ROM或SD卡，性能自然提高不少，对于Android手机而言同时还减少了电量消耗。该类实现的功能不是很多，直接从Object上继承，通过JNI的方式直接在C底层执行。

将已经存在的内存资源重新使用而避免去创建新的，最典型的使用就是缓存（Cache）和池（Pool）。

### **1. 节制地使用Service，**为了能够控制Service的生命周期，Android官方推荐的最佳解决方案就是使用**[IntentService](http://developer.android.com/reference/android/app/IntentService.html)**，这种Service的最大特点就是当后台任务执行结束后会自动停止，从而极大程度上避免了Service内存泄漏的可能性。

只需要在Activity中重写onTrimMemory\(\)方法，然后在这个方法中监听TRIM\_MEMORY\_UI\_HIDDEN这个级别，一旦触发了之后就说明用户已经离开了我们的程序，那么此时就可以进行资源释放操作了，在内存紧张的时候同样可以通过这个方法释放资源。

可能一张图片只有100k你觉得它并不大，但是读取到内存当中是按照像素点来算的，比如这张图片是1500\_1000像素，使用的ARGB\_8888颜色类型，那么每个像素点就会占用4个字节，总内存就是1500\_1000\*4字节，也就是5.7M，这个数据看起来就比较恐怖了。

### **使用ProGuard来剔除不需要的代码**

### **为序列化的数据使用nano protobufs**

### **对最终的APK使用zipalign**

在编写完所有代码，并通过编译系统生成APK之后，你需要使用[zipalign](http://developer.android.com/tools/help/zipalign.html)对APK进行重新校准。如果你不做这个步骤，会导致你的APK需要更多的RAM，因为一些类似图片资源的东西不能被mapped。

**Notes::**Google Play不接受没有经过zipalign的APK。





