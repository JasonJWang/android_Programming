## 如何单独编译Android源代码中的模块 ##

&nbsp;&nbsp;一. 首先在Android源代码目录下的build目录下，有个脚本文件envsetup.sh，执行这个脚本文件后，就可以获得一些有用的工具：
```bash
USER-NAME@MACHINE-NAME:~/Android$ .  ./build/envsetup.sh
```
&nbsp;&nbsp;注意，这是一个source命令，执行之后，就会有一些额外的命令可以使用：
```bash
- croot: Changes directory to the top of the tree.

- m: Makes from the top of the tree.
- mm: Builds all of the modules in the current directory.
- mmm: Builds all of the modules in the supplied directories.
- cgrep: Greps on all local C/C++ files.
- jgrep: Greps on all local Java files.
- resgrep: Greps on all local res/*.xml files.
- godir: Go to the directory containing a file.
```
&nbsp;&nbsp;这些命令的具体用法，可以在命令的后面加-help来查看，这里我们只关注mmm命令，也就是可以用它来编译指定目录的所有模块，通常这个目录只包含一个模块。

&nbsp;&nbsp;二. 使用mmm命令来编译指定的模块，例如Email应用程序：
```bash
USER-NAME@MACHINE-NAME:~/Android$ mmm packages/apps/Email/
```
&nbsp;&nbsp;编译完成之后，就可以在`out/target/product/generic/system/app`目录下看到Email.apk文件了。Android系统自带的App都放在这具目录下。另外，Android系统的一些可执行文件，例如C编译的可执行文件，放在`out/target/product/generic/system/bin`目录下，动态链接库文件放在`out/target/product/generic/system/lib`目录下，`out/target/product/generic/system/lib/hw`目录存放的是硬件抽象层（HAL）接口文件。
&nbsp;&nbsp;三. 编译好模块后，还要重新打包一下system.img文件，这样我们把system.img运行在模拟器上时，就可以看到我们的程序了。
```bash
USER-NAME@MACHINE-NAME:~/Android$ make snod
```
&nbsp;&nbsp;四. 参照Ubuntu上下载、编译和安装Android最新源代码一文介绍的方法运行模拟器：
```bash
USER-NAME@MACHINE-NAME:~/Android$ emulator
```
&nbsp;&nbsp;这样一切就搞定了。
&nbsp;&nbsp;