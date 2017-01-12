## 在Ubuntu上为Android增加硬件抽象层（HAL）模块访问Linux内核驱动程序 ##

&nbsp;&nbsp;一. 参照在Ubuntu上为Android系统编写Linux内核驱动程序一文所示，准备好示例内核驱动序。完成这个内核驱动程序后，便可以在Android系统中得到三个文件，分别是/dev/hello、/sys/class/hello/hello/val和/proc/hello。在本文中，我们将通过设备文件/dev/hello来连接硬件抽象层模块和Linux内核驱动程序模块。

&nbsp;&nbsp;二. 进入到在hardware/libhardware/include/hardware目录，新建hello.h文件：
```bash
USER-NAME@MACHINE-NAME:~/Android$ cd hardware/libhardware/include/hardware
USER-NAME@MACHINE-NAME:~/Android/hardware/libhardware/include/hardware$ vi hello.h
```

&nbsp;&nbsp;hello.h文件的内容如下：
```c
#ifndef ANDROID_HELLO_INTERFACE_H  
#define ANDROID_HELLO_INTERFACE_H  
#include <hardware/hardware.h>  
  
__BEGIN_DECLS  
  
/*定义模块ID*/  
#define HELLO_HARDWARE_MODULE_ID "hello"  
  
/*硬件模块结构体*/  
struct hello_module_t {  
    struct hw_module_t common;  
};  
  
/*硬件接口结构体*/  
struct hello_device_t {  
    struct hw_device_t common;  
    int fd;  
    int (*set_val)(struct hello_device_t* dev, int val);  
    int (*get_val)(struct hello_device_t* dev, int* val);  
};  
  
__END_DECLS  
  
#endif  
```

&nbsp;&nbsp;这里按照Android硬件抽象层规范的要求，分别定义模块ID、模块结构体以及硬件接口结构体。在硬件接口结构体中，fd表示设备文件描述符，对应我们将要处理的设备文件"/dev/hello"，set_val和get_val为该HAL对上提供的函数接口。

&nbsp;&nbsp;三. 进入到`hardware/libhardware/modules`目录，新建hello目录，并添加hello.c文件。 hello.c的内容较多，我们分段来看。

&nbsp;&nbsp;首先是包含相关头文件和定义相关结构：

```c
#define LOG_TAG "HelloStub"  
  
#include <hardware/hardware.h>  
#include <hardware/hello.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <cutils/log.h>  
#include <cutils/atomic.h>  
  
#define DEVICE_NAME "/dev/hello"  
#define MODULE_NAME "Hello"  
#define MODULE_AUTHOR "shyluo@gmail.com"  
  
/*设备打开和关闭接口*/  
static int hello_device_open(const struct hw_module_t* module, const char* name, struct hw_device_t** device);  
static int hello_device_close(struct hw_device_t* device);  
  
/*设备访问接口*/  
static int hello_set_val(struct hello_device_t* dev, int val);  
static int hello_get_val(struct hello_device_t* dev, int* val);  
  
/*模块方法表*/  
static struct hw_module_methods_t hello_module_methods = {  
    open: hello_device_open  
};  
  
/*模块实例变量*/  
struct hello_module_t HAL_MODULE_INFO_SYM = {  
    common: {  
        tag: HARDWARE_MODULE_TAG,  
        version_major: 1,  
        version_minor: 0,  
        id: HELLO_HARDWARE_MODULE_ID,  
        name: MODULE_NAME,  
        author: MODULE_AUTHOR,  
        methods: &hello_module_methods,  
    }  
};  
```
&nbsp;&nbsp;这里，实例变量名必须为HAL_MODULE_INFO_SYM，tag也必须为HARDWARE_MODULE_TAG，这是Android硬件抽象层规范规定的。

&nbsp;&nbsp;定义hello_device_open函数：

```c
static int hello_device_open(const struct hw_module_t* module, const char* name, struct hw_device_t** device) {  
    struct hello_device_t* dev;dev = (struct hello_device_t*)malloc(sizeof(struct hello_device_t));  
      
    if(!dev) {  
        LOGE("Hello Stub: failed to alloc space");  
        return -EFAULT;  
    }  
  
    memset(dev, 0, sizeof(struct hello_device_t));  
    dev->common.tag = HARDWARE_DEVICE_TAG;  
    dev->common.version = 0;  
    dev->common.module = (hw_module_t*)module;  
    dev->common.close = hello_device_close;  
    dev->set_val = hello_set_val;dev->get_val = hello_get_val;  
  
    if((dev->fd = open(DEVICE_NAME, O_RDWR)) == -1) {  
        LOGE("Hello Stub: failed to open /dev/hello -- %s.", strerror(errno));free(dev);  
        return -EFAULT;  
    }  
  
    *device = &(dev->common);  
    LOGI("Hello Stub: open /dev/hello successfully.");  
  
    return 0;  
}  
```

&nbsp;&nbsp;DEVICE_NAME定义为"/dev/hello"。由于设备文件是在内核驱动里面通过device_create创建的，而device_create创建的设备文件默认只有root用户可读写，而hello_device_open一般是由上层APP来调用的，这些APP一般不具有root权限，这时候就导致打开设备文件失败：
```
Hello Stub: failed to open /dev/hello -- Permission denied.
```
&nbsp;&nbsp;解决办法是类似于Linux的udev规则，打开Android源代码工程目录下，进入到system/core/rootdir目录，里面有一个名为ueventd.rc文件，往里面添加一行：
```bash
 /dev/hello 0666 root root
```
&nbsp;&nbsp;定义hello_device_close、hello_set_val和hello_get_val这三个函数：
```c
static int hello_device_close(struct hw_device_t* device) {  
    struct hello_device_t* hello_device = (struct hello_device_t*)device;  
  
    if(hello_device) {  
        close(hello_device->fd);  
        free(hello_device);  
    }  
      
    return 0;  
}  
  
static int hello_set_val(struct hello_device_t* dev, int val) {  
    LOGI("Hello Stub: set value %d to device.", val);  
  
    write(dev->fd, &val, sizeof(val));  
  
    return 0;  
}  
  
static int hello_get_val(struct hello_device_t* dev, int* val) {  
    if(!val) {  
        LOGE("Hello Stub: error val pointer");  
        return -EFAULT;  
    }  
  
    read(dev->fd, val, sizeof(*val));  
  
    LOGI("Hello Stub: get value %d from device", *val);  
  
    return 0;  
}  
```
&nbsp;&nbsp;四. 继续在hello目录下新建Android.mk文件：
```bash
      LOCAL_PATH := $(call my-dir)
      include $(CLEAR_VARS)
      LOCAL_MODULE_TAGS := optional
      LOCAL_PRELINK_MODULE := false
      LOCAL_MODULE_PATH := $(TARGET_OUT_SHARED_LIBRARIES)/hw
      LOCAL_SHARED_LIBRARIES := liblog
      LOCAL_SRC_FILES := hello.c
      LOCAL_MODULE := hello.default
      include $(BUILD_SHARED_LIBRARY)
```
&nbsp;&nbsp;注意，LOCAL_MODULE的定义规则，hello后面跟有default，hello.default能够保证我们的模块总能被硬象抽象层加载到。

&nbsp;&nbsp;五. 编译：
```bash
USER-NAME@MACHINE-NAME:~/Android$ mmm hardware/libhardware/modules/hello
```
&nbsp;&nbsp;编译成功后，就可以在`out/target/product/generic/system/lib/hw`目录下看到`hello.default.so`文件了。
&nbsp;&nbsp;六. 重新打包Android系统镜像system.img：
```bash
USER-NAME@MACHINE-NAME:~/Android$ make snod
```
&nbsp;&nbsp;重新打包后，system.img就包含我们定义的硬件抽象层模块hello.default了。
  虽然我们在Android系统为我们自己的硬件增加了一个硬件抽象层模块，但是现在Java应用程序还不能访问到我们的硬件。我们还必须编写JNI方法和在Android的Application Frameworks层增加API接口，才能让上层Application访问我们的硬件。在接下来的文章中，我们还将完成这一系统过程，使得我们能够在Java应用程序中访问我们自己定制的硬件。
