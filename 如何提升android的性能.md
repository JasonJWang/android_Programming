## 如何提升Android应用的性能 ##

### 第一个要点：首先要有良好的编程习惯 ###
要成为一名优秀的资源管理员;既要运用常识，还要使用公认的算法和标准的设计模式。在资源使用方面，如果你打开了资源，要记得关闭资源。要尽量晚地获取，尽量早地释放。这些由来已久的编程准则同样适用于你的Android应用程序，如果它们使用底层的设备服务，更是如此。

### 第二个要点：让阻塞操作远离主用户界面线程 ###
想确保你的应用程序运行起来很灵活，就要使用AsyncTask、线程、IntentService或自定义后台服务来处理脏活。应使用装入器来简化装入时间长的数据(如游标)的状态管理。你无法容忍你的应用程序在某个操作正在处理的时候出现滞后或停顿。

### 第三个要点：使用最新的Android软件开发工具包(SDK)版本、应用编程接口(API)和最佳实践 ###
确保你开发的应用程序是最新的，因而要使用Android平台提供的最新工具。随着Android平台不断发展，它也在不断改进。一些功能 可能已被弃用，或者换成了更好的功能。核心API得到了修正版(bug fix)和性能改进。已经引入了装入器等新的API，帮助开发者编写出运行更稳定、响应更迅即的应用程序。

### 第四个要点：考虑使用限制模式(Strict Mode) ＃＃＃
你可以使用名为限制模式(StrictMode)的AndroidAPI，帮助你查明哪里违反了几个良好的编程习惯。StrictMode 会帮助你确认你的应用程序是不是存在内存泄漏，并且检测你的应用程序是不是在试图执行长时间的阻塞操作，这些操作应该被卸载到线程或别的渠道。
Android2.3里面引入StrictMode类(android.os.StrictMode)。

### 第五个要点：在发布应用程序之前，禁用或尽量少用调试和诊断　###
如果你的Android应用程序开发起来需要一些时间，你可能已将一些日志和调试代码嵌入到了应用程序中。写入到日志及其他此类输出系统给性能带来了影响。确保在发布应用程序之前，尽量少用或完全禁用这些功能。

### 第六个要点：确保你设计的布局简单、简练和浅层 ###

简单的屏幕有助于阅读起来最轻松，而简单的布局装入起来最快速。你不应该过于深层地嵌套你的布局，或者用不必要的过多视图(View)控件 塞满屏幕。花些时间来开发用户可以高效使用的简练用户界面，而不是试图把太多功能塞入到单单一个屏幕上。这不但有助于提升应用程序的性能，还有助于让你的 应用程序对用户来说更高效。

### 第七个要点：让你应用程序的资源适合目标设备 ###
添加适合特定设备配置的资源，那样它们就能尽可能高效地装入。我们在谈论图形资源时，这点尤为重要。如果你添加了可利用的庞大图像资源，需要装入和调整大小，就无法有效地使用其他的应用程序资源。

### 第八个要点：使用Hierarchy Viewer工具 ###
Hierarchy Viewer工具可以帮助你调试你的应用程序布局。它还提供了宝贵的分析信息，以便了解布局里面的每一个视图控件测量、渲染和绘制要花多少时间。只有准确找到了问题的根源，问题解决起来才容易。

### 第九个要点：使用layoutopt工具 ###
Layoutopt工具是一款简单的命令行工具，它可以帮助你找到不必要的控件嵌套以及缩减布局资源的其他方法，以便尽量减少资源的使用。它让你可以了解哪些布局控件可能是多余的或不必要的。控件越少、布局层次越浅，性能就越好。

### 总结 ###

性能是移动应用的关键。谷歌提供了许多提升Android应用性能的培训资料。 简单概括了其中的技巧和技术。
作为提升性能的一个综合方法，需要做好以下几个方面的基础工作：

* 内存管理 
* 语言特性和库的使用 
* 布局层次 
* 电池续航 
* 多线程 
* UI响应能力 

将这些资料全部掌握非常困难，因此，写过多本Android开发相关书籍的Shane Conder和Lauren Darcey提出了几项建议，包括：

* 运用良好的编码实践
* 不要使用主线程阻塞操作 
* 确保布局简单优雅 
* 针对设备裁剪资源 
* 使用traceview和其它性能分析工具 

在开发应用时，这些建议可以预先提供帮助。但当应用几乎已经开发完成，却发现性能不尽人意，该怎么办呢？Tutsplus的Jessica Thornsby指出了三个容易检查的关键方面：

  1.“过度绘制（overdraw）”：这在GPU绘制完背景后再在上面绘制其它图形工件时就会发生。过度绘制太多会杀死应用的性能。Android支持“GPU过度绘制调试”模式，高亮过度绘制区域，使用不同的颜色展示重复绘制的次数。这样，开发者就有了线索，知道应该修改哪个区域。 

  2.渲染管道：UI渲染耗时与层次中的视图数量成正比。Android SDK层次查看器可以帮助开发者检查视图层次，找出将其扁平化的方法。它还提供了有关每个视图渲染消耗多少时间的性能分析信息。 

  3.内存泄露：尽管有垃圾收集，但Android并不能免于内存泄露。Android Studio中内置的Android内存监视器允许开发者查看应用使用了多少内存。Android设备监视器的Heap选项卡是另一款有助于解决内存泄露的工具。关于哪个对象真正地占用了设备内存，它提供了更多细节信息。 


最后，简化应用UI被认为是Jeannie Liou提出的最重要的性能提升建议。他指出，在组织视图时，使用相对布局要优于线性布局。