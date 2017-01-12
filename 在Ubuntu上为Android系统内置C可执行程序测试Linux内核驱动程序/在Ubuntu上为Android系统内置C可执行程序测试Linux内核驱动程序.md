## 在Ubuntu上为Android系统内置C可执行程序测试Linux内核驱动程序 ##

&nbsp;&nbsp;一. 准备好Linux驱动程序。使用Android模拟器加载包含这个Linux驱动程序的内核文件，并且使用adb shell命令连接上模拟，验证在/dev目录中存在设备文件hello。

&nbsp;&nbsp;二. 进入到Android源代码工程的external目录，创建hello目录:

```bash
USER-NAME@MACHINE-NAME:~/Android$ cd external
USER-NAME@MACHINE-NAME:~/Android/external$ mkdir hello
```

&nbsp;&nbsp;三. 在hello目录中新建hello.c文件：

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <fcntl.h>  
#define DEVICE_NAME "/dev/hello"  
int main(int argc, char** argv)  
{  
    int fd = -1;  
    int val = 0;  
    fd = open(DEVICE_NAME, O_RDWR);  
    if(fd == -1) {  
        printf("Failed to open device %s.\n", DEVICE_NAME);  
        return -1;  
    }  
      
    printf("Read original value:\n");  
    read(fd, &val, sizeof(val));  
    printf("%d.\n\n", val);  
    val = 5;  
    printf("Write value %d to %s.\n\n", val, DEVICE_NAME);  
        write(fd, &val, sizeof(val));  
      
    printf("Read the value again:\n");  
        read(fd, &val, sizeof(val));  
        printf("%d.\n\n", val);  
    close(fd);  
    return 0;  
}  
```

&nbsp;&nbsp;这个程序的作用中，打开/dev/hello文件，然后先读出/dev/hello文件中的值，接着写入值5到/dev/hello中去，最后再次读出/dev/hello文件中的值，看看是否是我们刚才写入的值5。从/dev/hello文件读写的值实际上就是我们虚拟的硬件的寄存器val的值

&nbsp;&nbsp;四. 在hello目录中新建Android.mk文件：
```bash
      LOCAL_PATH := $(call my-dir)

      include $(CLEAR_VARS)

      LOCAL_MODULE_TAGS := optional

      LOCAL_MODULE := hello

      LOCAL_SRC_FILES := $(call all-subdir-c-files)

      include $(BUILD_EXECUTABLE)
```

&nbsp;&nbsp;注意，BUILD_EXECUTABLE表示我们要编译的是可执行程序。 

&nbsp;&nbsp;五. 使用mmm命令进行编译：
```bash
USER-NAME@MACHINE-NAME:~/Android$ mmm ./external/hello
```
&nbsp;&nbsp;编译成功后，就可以在`out/target/product/gerneric/system/bin`目录下，看到可执行文件hello了。

&nbsp;&nbsp;六. 重新打包Android系统文件system.img：
```bash
USER-NAME@MACHINE-NAME:~/Android$ make snod
```
&nbsp;&nbsp;这样，重新打包后的system.img文件就包含刚才编译好的hello可执行文件了。

&nbsp;&nbsp;七. 运行模拟器，使用`/system/bin/hello`可执行程序来访问Linux内核驱动程序：
```bash
USER-NAME@MACHINE-NAME:~/Android$ emulator -kernel ./kernel/common/arch/arm/boot/zImage &

USER-NAME@MACHINE-NAME:~/Android$ adb shell

root@android:/ # cd system/bin

root@android:/system/bin # ./hello

Read the original value:

0.

Write value 5 to /dev/hello.

Read the value again:

5.
```
  看到这个结果，就说我们编写的C可执行程序可以访问我们编写的Linux内核驱动程序了。