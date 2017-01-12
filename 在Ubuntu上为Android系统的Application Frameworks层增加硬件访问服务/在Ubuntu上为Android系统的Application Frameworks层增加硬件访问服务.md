## 在Ubuntu上为Android系统的Application Frameworks层增加硬件访问服务 ##

&nbsp;&nbsp;一. 为硬件抽象层模块准备好JNI方法调用层。

&nbsp;&nbsp;二. 在Android系统中，硬件服务一般是运行在一个独立的进程中为各种应用程序提供服务。因此，调用这些硬件服务的应用程序与这些硬件服务之间的通信需要通过代理来进行。为此，我们要先定义好通信接口。进入到`frameworks/base/core/java/android/os`目录，新增`IHelloService.aidl`接口定义文件：
```bash
USER-NAME@MACHINE-NAME:~/Android$ cd frameworks/base/core/java/android/os
USER-NAME@MACHINE-NAME:~/Android/frameworks/base/core/java/android/os$ vi IHelloService.aidl
```

&nbsp;&nbsp;`IHelloService.aidl`定义了IHelloService接口：
```java
package android.os;  
   
interface IHelloService {  
    void setVal(int val);  
    int getVal();  
}  
```
&nbsp;&nbsp;IHelloService接口主要提供了设备和获取硬件寄存器val的值的功能，分别通过setVal和getVal两个函数来实现。

&nbsp;&nbsp;三.返回到`frameworks/base`目录，打开Android.mk文件，修改LOCAL_SRC_FILES变量的值，增加IHelloService.aidl源文件：

```bash
## READ ME: ########################################################

   ##

   ## When updating this list of aidl files, consider if that aidl is

   ## part of the SDK API. If it is, also add it to the list below that

   ## is preprocessed and distributed with the SDK. This list should

   ## not contain any aidl files for parcelables, but the one below should

   ## if you intend for 3rd parties to be able to send those objects

   ## across process boundaries.

   ##

   ## READ ME: ########################################################

   LOCAL_SRC_FILES += /

   ....................................................................

   core/java/android/os/IVibratorService.aidl /

   core/java/android/os/IHelloService.aidl /

   core/java/android/service/urlrenderer/IUrlRendererService.aidl /

   .....................................................................
```
&nbsp;&nbsp;四. 编译`IHelloService.aidl`接口：
```bash
USER-NAME@MACHINE-NAME:~/Android$ mmm frameworks/base
```
&nbsp;&nbsp;这样，就会根据`IHelloService.aidl`生成相应的`IHelloService.Stub`接口。

&nbsp;&nbsp;五.进入到`frameworks/base/services/java/com/android/server`目录，新增`HelloService.java`文件：
```java
package com.android.server;  
import android.content.Context;  
import android.os.IHelloService;  
import android.util.Slog;  
public class HelloService extends IHelloService.Stub {  
    private static final String TAG = "HelloService";  
    HelloService() {  
        init_native();  
    }  
    public void setVal(int val) {  
        setVal_native(val);  
    }     
    public int getVal() {  
        return getVal_native();  
    }  
      
    private static native boolean init_native();  
        private static native void setVal_native(int val);  
    private static native int getVal_native();  
};  
```

&nbsp;&nbsp;HelloService主要是通过调用JNI方法init_native、setVal_native和getVal_native来提供硬件服务。

&nbsp;&nbsp;六. 修改同目录的`SystemServer.java`文件，在`ServerThread::run`函数中增加加载HelloService的代码：
```java
@Override

public void run() {

  ......

  try {

    Slog.i(TAG, "DiskStats Service");

    ServiceManager.addService("diskstats", new DiskStatsService(context));

  } catch (Throwable e) {

    Slog.e(TAG, "Failure starting DiskStats Service", e);

  }

  try {

    Slog.i(TAG, "Hello Service");

    ServiceManager.addService("hello", new HelloService());

  } catch (Throwable e) {

    Slog.e(TAG, "Failure starting Hello Service", e);

  }

  .........

}      
```

&nbsp;&nbsp;七. 编译HelloService和重新打包system.img：
```bash
USER-NAME@MACHINE-NAME:~/Android$ mmm frameworks/base/services/java
USER-NAME@MACHINE-NAME:~/Android$ make snod
```
  这样，重新打包后的system.img系统镜像文件就在Application Frameworks层中包含了我们自定义的硬件服务HelloService了，并且会在系统启动的时候，自动加载HelloService。这时，应用程序就可以通过Java接口来访问Hello硬件服务了。
&nbsp;&nbsp;
