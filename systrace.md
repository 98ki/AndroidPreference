## **Systrace简单介绍**

Systrace是Android4.1中新增的性能数据采样和分析工具。它可帮助开发者收集Android关键子系统（如surfaceflinger、WindowManagerService等Framework部分关键模块、服务，View系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

Systrace的功能包括跟踪系统的I\/O操作、内核工作队列、CPU负载以及Android各个子系统的运行状况等。在Android平台中，它主要由3部分组成：

* 内核部分：Systrace利用了Linux Kernel中的ftrace功能。所以，如果要使用Systrace的话，必须开启kernel中和ftrace相关的模块。
* 数据采集部分：Android定义了一个Trace类。应用程序可利用该类把统计信息输出给ftrace。同时，Android还有一个atrace程序，它可以从ftrace中读取统计信息然后交给数据分析工具来处理。
* 数据分析工具：Android提供一个systrace.py（python脚本文件，位于Android SDK目录\/tools\/systrace中，其内部将调用atrace程序）用来配置数据采集的方式（如采集数据的标签、输出文件名等）和收集ftrace统计数据并生成一个结果网页文件供用户查看。 从本质上说，Systrace是对Linux Kernel中ftrace的封装。应用进程需要利用Android提供的Trace类来使用Systrace.

  关于Systrace的官方介绍和使用可以看这里：[Systrace](http://developer.android.com/tools/help/systrace.html "SysTrace官方介绍")

## **Systrace简单使用**

使用Systrace前，要先了解一下Systrace在各个平台上的使用方法，鉴于大家使用Eclipse和Android Studio的居多，所以直接摘抄官网关于这个的使用方法，不过不管是什么工具，流程是一样的：

* 手机准备好你要进行抓取的界面
* 点击开始抓取（命令行的话就是开始执行命令）
* 手机上开始操作
* 设定好的时间到了之后，会将生成Trace文件，使用Chrome将这个文件打开进行分析

#### **Using Eclipse**

1. In Eclipse, open an Android application project.

  1. Switch to the DDMS perspective, by selecting Window &gt; Perspectives &gt; DDMS.
  2. In the Devices tab, select the device on which to run a trace. If no devices are listed, make sure your device is connected via USB cable and that debugging is enabled on the device.
  3. Click the Systrace icon at the top of the Devices panel to configure tracing.
  4. Set the tracing options and click OK to start the trace.


#### **Using Android Studio**

1. In Android Studio, open an Android application project.

  1. Open the Device Monitor by selecting Tools &gt; Android &gt; Monitor.
  2. In the Devices tab, select the device on which to run a trace. If no devices are listed, make sure your device is connected via USB cable and that debugging is enabled on the device.
  3. Click the Systrace icon at the top of the Devices panel to configure tracing.
  4. Set the tracing options and click OK to start the trace.


#### **Using Device Monitor**

1. Navigate to your SDK tools\/ directory.

  1. Run the monitor program.
  2. In the Devices tab, select the device on which to run a trace. If no devices are listed, make sure your device is connected via USB cable and that debugging is enabled on the device.
  3. Click the Systrace icon at the top of the Devices panel to configure tracing.
  4. Set the tracing options and click OK to start the trace.


#### **Command Line Usage**

命令行形式比较灵活，速度也比较快，一次性配置好之后，以后再使用的时候就会很快就出结果（**强烈推荐**）

| 12 | $ cd android-sdk\/platform-tools\/systrace$ python systrace.py --time=10 -o mynewtrace.html sched gfx view wm |
| --- | --- |

从上面的命令可以看到Systrace工具的位置，只需要在Bash中配置好对应的路径和Alias，使用起来还是很快速的。另外User版本是不可以抓Trace的，只有ENG版本或者Userdebug版本才可以。

抓取结束后，会生成对应的Trace文件，注意这个文件只能被Chrome打开。关于如何分析Trace文件，我们下面的章节会讲。不论使用那种工具，在抓取之前都会让选择参数，下面说一下这些参数的意思：

* -h, –help Show the help message.（帮助）
* -o Write the HTML trace report to the specified file.（即输出文件名，）
* -t N, –time=N Trace activity for N seconds. The default value is 5 seconds. （Trace抓取的时间，一般是 ： -t 8）
* -b N, –buf-size=N Use a trace buffer size of N kilobytes. This option lets you limit the total size of the data collected during a trace.
* -k
* —ktrace= Trace the activity of specific kernel functions, specified in a comma-separated list.
* -l, –list-categories List the available tracing category tags. The available tags are\(下面的参数不用翻译了估计大家也看得懂，贴官方的解释也会比较权威，后面分析的时候我们会看到这些参数的作业的\):

  * **gfx** - Graphics
  * **input** - Input
  * **view** - View
  * webview - WebView
  * **wm** - Window Manager
  * **am** - Activity Manager
  * audio - Audio
  * video - Video
  * camera - Camera
  * hal - Hardware Modules
  * res - Resource Loading
  * **dalvik** - Dalvik VM
  * rs - RenderScript
  * **sched** - CPU Scheduling
  * **freq** - CPU Frequency
  * **membus** - Memory Bus Utilization
  * **idle** - CPU Idle
  * **disk** - Disk input and output
  * **load** - CPU Load
  * **sync** - Synchronization Manager
  * **workq** - Kernel Workqueues Note: Some trace categories are not supported on all devices. Tip: If you want to see the names of tasks in the trace output, you must include the sched category in your command parameters.

* -a

* —app= Enable tracing for applications, specified as a comma-separated list of package names. The apps must contain tracing instrumentation calls from the Trace class. For more information, see Analyzing Display and Performance.
* —link-assets Link to the original CSS or JavaScript resources instead of embedding them in the HTML trace report.
* —from-file= Create the interactive Systrace report from a file, instead of running a live trace.
* —asset-dir= Specify a directory for the trace report assets. This option is useful for maintaining a single set of assets for multiple Systrace reports.
* -e
* —serial= Conduct the trace on a specific connected device, identified by its device serial number.

  上面的参数虽然比较多，但使用工具的时候不需考虑这么多，在对应的项目前打钩即可，命令行的时候才会去手动加参数：

我们一般会把这个命令配置成Alias，配置如下：

| 12 | alias st-start='python \/home\/gaojianwu\/Software\/android-studio\/sdk\/platform-tools\/systrace\/systrace.py'  alias st-start-gfx-trace = ‘st-start -t 8 gfx input view sched freq wm am hwui workq res dalvik sync disk load perf hal rs idle mmc’ |
| --- | --- |

这样在使用的时候，可以直接敲 **st-start-gfx-mx4** 即可，当然为了区分和保持各个文件，还需要加上 **-o xxx.Trace** .上面的命令和参数不必一次就理解，只需要记住如何简单使用即可，在分析的过程中，这些东西都会慢慢熟悉的。

