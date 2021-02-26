# JVM监控及诊断工具-命令行

## 概述

Java作为最流行的编程语言之一，其应用性能诊断一直受到业界广泛关注。可能造成Java引用出现性能问题的因素很多，比如线程控制、磁盘读写、数据库访问、网络I/O、垃圾收集等。想要定位这些问题。一款优秀的性能诊断工具必不可少。

## jps：查看正在运行的Java进程

jps（Java Process Status）：显示指定系统内所有的HotSpot虚拟机进程（查看虚拟机进程信息），可用于查询正在运行的虚拟机进程。

说明：对于本地虚拟机进程来说，进程的本地虚拟机ID与操作系统的进程ID是一致的，是唯一的。

### 基本语法

```shell
jps [options] [hostid]
```

#### options参数

-q：仅仅显示LVMID（local vertual machine id），即本地虚拟机唯一id。不显示主类的名称等

-l：输出应用程序主类的全类名或如果进程执行的是jar包，则输出jar完整路径

-m：输出虚拟机进程启动时传递给主类main()的参数

-v：列出虚拟机进程启动时的JVM参数。

### hostid参数

RMI注册表中注册的主机名。

如果想要远程监控主机上的java程序，需要安装jstatd。

## jstat：查看JVM统计信息

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据。

在没有GUI图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。

### 基本语法

```shell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

#### option参数

- 类装载相关的：
  - -class：显示ClassLoader的相关信息：类的装载、卸载数量、总空间、类装载所消耗的时间等
- 垃圾回收相关的：
  - -gc：显示与GC相关的堆信息。包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息。
  - -gccapacity：显示内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间。
  - -gcutil：显示内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比。
  - -gccause：与-gcutil功能一样，但是会额外输出导致最后一次货当前正在发生的GC产生的原因。
  - -gcnew：显示新生代GC状况。
  - -gcnewcapacity：显示内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间。
  - -geold：显示老年代GC状况。
  - -geoldcapacity：显示内容与-geold基本相同，输出主要关注使用到的最大、最小空间。
- JIT相关的：
  - -complier：显示JIT编译器编译过的方法、耗时等信息
  - -printcompilation：输出已经被JIT编译的方法

#### interval参数

用于指定输出统计数据的周期，单位为毫秒，即：查询间隔

#### count参数

用于指定查询的总次数

#### -t参数

可以在输出信息前加上一个TimeStamp列，显示程序的运行时间。单位：秒

> 我们可以比较Java进程的启动时间以及总GC时间（GCT列），或者两次测量的间隔时间以及总GC时间的增量，来得出GC时间占运行时间的比例。
>
> 如果该比例超过20%，则说明目前堆的压力较大；如果该比例超过90%，则说明堆里几乎没有可用空间，随时都可能抛出OOM异常。

#### -h参数

可以在周期性数据输出时，输出多少行数据后输出一个表头信息

#### 补充

jstat还可以用来判断是否出现内存泄漏

第1步：在长时间运行的Java程序中，我们可以运行jstat命令连续获取多行性能数据，并取这几行数据中OU列（即已占用的老年代内存）的最小值。

第2步：然后，我们每隔一段较长的时间重复一次上述操作，来获得多组OU最小值。如果这些值成上涨趋势，则说明该Java程序的老年代内存已使用量在不断上涨，这意味着无法回收的对象在不断增加，因此很有可能存在内存泄漏。

## jinfo：实时查看和修改JVM配置参数

jinfo（Configuration Info for Java）查看虚拟机配置参数信息，也可用于调整虚拟机的配置参数。

### 基本语法

```shell
jinfo [options] pid #java进程ID必须要加上
```

[options]：

| 选项             | 选项说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| no option        | 输出全部的参数和系统属性                                     |
| -flag name       | 输出对应名称的参数                                           |
| -flag [+-] name  | 开启或者关闭对应名称的参数，只有被标记为manageable的参数才可以被动态修改 |
| -flag name=value | 设定对应名的参数                                             |
| -flags           | 输出全部的参数                                               |
| -sysprops        | 输出系统属性                                                 |

#### 查看：

jinfo -sysprops PID：可以查看由System.getProperties()取得的参数

jinfo -flags PID：查看曾经赋过值的一些参数

jinfo -flag 具体参数 PID：查看某个java进程的具体参数的值

#### 修改：

jinfo不仅可以查看运行时某一个Java虚拟机参数的实际取值，甚至可以在运行时修改部分参数，并使之立即生效。

但是，并非所有参数都支持动态修改。参数只有被标记为manageable的flag可以被实时修改。其实，这个修改能力是极其有限的。

```shell
# 可以查看被标记为manageable的参数
java -XX:+PrintFlagsFinal -version | grep manageable
```

针对boolean类型：jinfo -flag [+|-]具体参数 PID

针对非boolean类型：jinfo -flag 具体参数=具体参数值 PID

### 拓展

java -XX:+printFlagsInitial：查看所有JVM参数启动的初始值

java -XX:+PrintFlagsFinal：查看所有JVM参数的最终值

java -XX:+PrintommandLineFlags：查看那些已经被用户或者JVM设置过的详细的XX参数的名称和值

## jmap：导出内存映像文件&内存使用情况

jmap（JVM Memory Map）：作用一方面是获取dump文件（堆转储快照文件，二进制文件），它还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况、堆中对象的统计信息、类加载信息等。

### 基本语法

```shell
jmap [option] <pid>
jmap [option] <executable <core>
jmap [option] [server_id@]<remote server IP or hostname>
```

其中option包括：

| 选项           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| -dump          | 生成dump文件                                                 |
| -finalizerinfo | 以ClassLoader为统计口径输出永久代的内存状态信息              |
| -heap          | 输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用信息等 |
| -histo         | 输出堆空间中对象的统计信息，包括类、实例数量和合计容量       |
| -permstat      | 以ClassLoader为统计口径输出永久代的内存状态信息              |
| -F             | 当虚拟机进程对-dump选项没有任何响应时，强制执行生成dump文件  |

-dump：

- 生成Java堆转储快照：dump文件
- 特别的：-dump:live只保存堆中的存活对象

-heap：

- 输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用等

-histo：

- 输出堆中对象的统计信息，包括类、实例数量和合计容量
- 特别的：histo:live只统计堆中的存活对象

-permstat

- 以ClassLoader为统计口径输出永久代的内存状态信息
- 仅linux/solaris平台有效

-finalizerinfo：

- 显示在F-Queue中等待Finalizer线程执行finalize方法的对象
- 仅linux/solaris平台有效

-F：

- 当虚拟机进程对-dump选项没有任何响应时，可使用此选项强制执行生成dump文件
- 仅linux/solaris平台有效

-h|-help：

- jmap工具使用的帮助命令

`-J <FLAG>`

- 传递参数给jmap启动的jvm

### 使用1：导出内存映像文件

#### 手动的方式

```bash
jmap -dump:format=b,file=<filename.hprof> <pid>
# live子参数只dump出存活的对象
jmap -dump:live,format=b,file=<filename.hprof> <pid>
```

#### 自动的方式

```bash
# 在程序发生OOM时，导出应用程序的当前堆快照
-XX:+HeapDumpOnOutOfMemoryError
# 指定堆快照的保存位置
-XX:HempDumpPath
```

### 使用2：显示退内存相关信息

```bash
jmap -heap pid
jmap -histo pid
```

### 使用3：其他作用

~~~bash
# 查看系统的ClassLoader信息
jmap -permstat pid
# 查看堆积在finalizer队列中的对象
jmap -finalizerinfo
~~~

## jhat：JDK自带堆分析工具

### 基本情况

jhat（JVM Heap Analysis Tool）：与jmap命令搭配使用，用于分析jmap生成的heap dump文件（堆转储快照）。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，用户可以在浏览器中查看分析结果（分析虚拟机转储快照信息）。

使用了jhat命令，就启动了一个http服务，端口是7000，即http://localhost:7000/，就可以在浏览器里分析。

说明：jhat命令在JDK9、JDK10中已经被删除，官方建议用VisualVM代替。

### 基本语法

~~~bash
jhat [option] xxx.hrof
~~~

## jstack：打印JVM中线程快照

### 基本情况

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者
javacore文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

生成线程快照的作用：可用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂起等，都是导致线程长时间停顿的常见原因。当线程出现停顿时，就可以使用jstack显示各个线程调用的堆栈情况。

在thread dump中，要留意下面几种状态

- 死锁，Deadlock（重点关注）
- 等待资源，Waiting on Condition（重点关注）
- 等待获取监视器，Waiting on monitor entry（重点关注）
- 阻塞，Blocked（重点关注）
- 执行中，Runnable
- 暂停，Suspended
- 对象等待中，Object.wait() 或 TIMED_WAITING
- 停止，Parked

### 基本语法

~~~bash
jstack option pid
~~~

| option参数 | 作用                                         |
| ---------- | -------------------------------------------- |
| -F         | 当正常输出的请求不被响应时，强制输出线程堆栈 |
| -l         | 除堆栈外，显示关于锁的附加信息               |
| -m         | 如果调用本地方法的话，可以显示C/C++的堆栈    |
| -h         | 帮助                                         |

## jcmd：多功能命令行

### 基本情况

在JDK1.7以后，新增了一个命令行工具jcmd。

它是一个多功能的工具，可以用来实现前面除了jstat之外所有命令的功能。jcmd拥有jmap的大部分功能，并且Oracle的官方网站上也推荐使用jcmd命令代替jmap命令。

### 基本语法

~~~bash
# 列出所有的JVM进程
jcmd -l
# 针对指定的进程，列出支持的所有命令
jcmd pid help
# 显示指定进程的指令命令的数据
jcmd pid 具体命令
~~~

## jstatd：远程主机信息收集

之前的指令只涉及到监控本机的Java应用程序，而这些工具中，一些监控工具也支持对远程计算机的监控（如jps、jstat）。为了启用远程监控，则需要配合使用jstatd工具。

命令jstatd是一个RMI服务端程序，它的作用相当于代理服务器，建立本地计算机与远程监控工具的通信。jstatd服务器将本地的Java应用程序信息传递到远程计算机。