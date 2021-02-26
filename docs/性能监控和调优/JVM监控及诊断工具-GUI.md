# JVM监控及诊断工具-GUI

## 工具概述

- JDK自带的工具
  - jconsole：JDK自带的可视化监控工具。查看Java应用程序的运行概况、监控堆信息、永久代（或元空间）使用情况、类加载情况等
  - Visual VM：提供了可是界面，用于查看Java虚拟机上运行的基于Java技术的应用程序的详细信息。
  - JMC：Java Mission Control，内置Java Flight Recorder。能够以极低的性能开销收集Java虚拟机的性能数据。
- 第三方工具
  - MAT：基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗
  - JProfiler：商业软件，需要付费。功能强大。
  - Arthas：Alibaba开源的Java诊断工具。
  - Btrace：Java运行时追踪工具。可以在不停机的情况下，跟踪指定的方法调用、构造函数调用和系统内存等信息。

## jConsole

### 基本概述

- 从JDK5开始，在JDK自带的java监控和管理控制台。
- 用于对JVM中内存、线程和类等的监控，是一个基于JMX（java management extensions）的GUI性能监控工具。

## Visual VM

### 基本概述

- 是一个功能强大的多合一故障诊断和性能监控的可视化工具。
- 集成了多个JDK命令行工具，使用Visual VM可用于显示虚拟机进程及进程的配置和环境信息（jps，jinfo），监视应用程序的CPU、GC、堆、方法区和线程的信息（jstat、jstack）等，甚至代替JConsole。
- Visual VM也可以作为独立的软件安装。

### 插件的安装

idea插件的安装、visualvm插件的安装

### 连接方式

#### 本地连接

监控本地Java进程的CPU、类、线程等

#### 远程连接

1. 确定远程服务器的ip地址
2. 添加JMX（通过JMX技术具体监控远端服务器哪个进程）
3. 修改bin/catalina.sh文件，连接远程的tocat
4. 在../conf添加jmxremote.access和jmxremote.password文件
5. 将服务器地址改为公网ip地址
6. 设置阿里云安全策略和防火墙cel
7. 启动tomcat，查看tomcat启动日志和端口监控
8. JMX中输入端口号、用户名、密码登录

### 主要功能

1. 生成/读取堆内存快照
2. 查看JVM参数和系统属性
3. 查看运行中的虚拟机进程
4. 生成/读取线程快照
5. 程序资源的实时监控
6. 其他功能（JMX代理连接、远程环境监控、CPU分析和内存分析）

## eclipse MAT

### 基本概述

MAT工具是一个款功能强大的Java堆内存分析器。可以用于查找内存泄漏以及查看内存消耗情况。

### 获取堆dump文件

#### dump文件内容

MAT可以分析heap dump文件。在进行内存分析时，只要获得了反应当前设备映像的hprof文件，通过MAT打开就可以直观地看到当前的内存信息。

一般来说，这些内存信息包含：

- 所有的对象信息，包括对象实例、成员变量、存储于栈中的基本类型值和存储于堆中的其它对象的引用值。
- 所有的类信息，包括classloader、类名称、父类、静态变量等
- GCRoot到所有的这些对象的引用路径
- 线程信息，包括线程的调用栈及此线程的线程局部变量（TLS）

#### 两点说明

1. 不是一个万能工具，并不能处理所有类型的堆存储文件，支持主流的厂家和格式。
2. 最吸引人的还是能够快速为开发人员生成内存泄漏报表，方便定位问题和分析问题。

#### 获取dump文件

1. jmap工具生成，可以生成任意一个java进程的dump文件
2. 通过配置参数生成
   - 选项`-XX:+HeapDumpOnOutOfMemoryError`或`-XX:+HeapDumpBeforeFullGC`
   - 选项`-XX:HeapDumpPath`表示当程序出现OOM时，将会在响应的目录下生成一份dump文件，若不指定该参数则在当前目录下生成dump文件
   - 考虑到生成环境中几乎不可能在线对其进行分析，大都是采用离线分析，因此使用jmap+MAT工具是最常见的组合。
3. 使用VisualVM可以导出dump文件
4. 使用MAT既可以打开衣蛾已有的堆快照，也可以通过MAT直接从活动Java程序中导出堆快照。

### 分析堆dump文件

#### histogram

MAT的直方图和jmap的-histo子命令一样，都能够展示各个类的实例数目以及这些实例的shallow heap总和。但是，MAT的直方图还能够计算Retained heap，并支持基于实例数目或Retained heap的排序方式（默认为Shallow heap）。

此外，MAT还可以将直方图中的类按照超类、类加载器或者包名分组。

当选中某个类时，MAT界面左上角的Inspector窗口将展示该类的Class实例的相关信息，如类加载器等。

展示了各个类的实例数目以及这些实例数目以及这些实例的Shallow heap或Retained heap的总和。

#### thread overview

- 查看系统中的Java线程
- 查看局部变量的信息

#### 获得对象相互引用的关系

- with outgoing references
- with incoming references

#### 浅堆与深堆

##### shallow heap

浅堆是指一个对象所消耗的内存。在32位系统中，一个对象引用会占用4个字节，一个int类型会占据4个字节，long类型变量会占据8个字节，每个对象头需要占用8个字节。根据堆快照格式不同，对象的大小可能会向8个字节进行对齐。

以String为例：2个int值共占8个字节，对象引用占用4个字节，对象头8字节，合计20字节，向8字节对齐，所以共占24字节。（JDK7）

这24字节为String对象的浅堆大小。它与String的value实际取值无关，无论字符串长度如何，浅堆大小始终是24字节。

##### retained heap

保留集（Retained Set）：对象A的保留集指当对象A被垃圾回收后，可以被释放的所有的对象集合（包括对象A本身），即对象A的保留集可以被认为只能通过对象A被直接或间接访问到的所有对象的集合。通俗的说，就是指仅被对象A所持有的对象的集合。

深堆是指对象的保留集中所有的对象的浅堆大小之和。

注意：浅堆指对象本身占用的内存，不包括其内部引用对象的大小。一个对象的深堆指只能通过该对象访问到的（直接或间接）所有对象的浅堆之和，即对象被回收后，可以释放的真实空间。

##### 补充：对象实际大小

另外一个常用的概念是对象的实际大小。这里，对象的实际大小定义为一个对象所能触及的所有对象的浅堆大小之和，也就是通常意义上我们说的对象大小。与深堆相比，这个在日常开发中更为直观和被人接受，但实际上，这个概念和垃圾回收无关。

下图显示了一个简单的对象引用关系图，对象A引用了C和D，对象B引用了C和E。那么对象A的浅堆大小只是A本身，不含C和D，而A的实际大小为A、C、D三者之和。而A的深堆大小为A和D之和，由于对象C还可以通过对象B访问到，因此不在对象A的深堆范围内。

![深堆和浅堆](https://github.com/jackhusky/jvm/blob/main/docs/images/深堆和浅堆.png)

#### 支配树

支配树（Dominator Tree）的概念源自图论。

MAT提供了一个称为支配树的对象图。支配树体现了对象实例间的支配关系。在对象引用图中，所有指向对象B的路径都经过对象A，则认为对象A支配对象B。如果对象A是离对象B最近的一个支配对象，则认为对象A为对象B的直接支配者。支配树是基于对象间的引用图所建立的，它有一下基本性质：

- 对象A的子树（所有被对象A支配的对象集合）表示对象A的保留集，即深堆。
- 如果对象A支持对象B，那么对象A的直接支配者也支配对象B。
- 支配树的边与对象引用图的边不直接对应。

如下图所示：左图表示对象引用图，右图表示左图对应的支配树。

![支配树](https://github.com/jackhusky/jvm/blob/main/docs/images/支配树.png)

## 再谈内存泄漏

### 内存泄漏的理解和分类

#### 内存泄漏（memory leak）的理解

严格来说，只有对象不会再被程序用到了，但是GC又不能回收它们的情况，才叫内存泄漏。

但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM，也可以叫做宽泛意义上的内存泄漏。

对象A引用对象Y，X的生命周期比Y的生命周期长。那么当Y生命周期结束的时候，X依然引用着Y，这时候，垃圾回收器不会回收对象Y的。

#### 内存泄漏和内存溢出的关系

1. 内存泄漏

   申请了内存用完了不释放，比如一共有1G的内存，分配了512M的内存一直不回收，那么可以用的内存只有512M了，仿佛泄漏掉了一部分。

2. 内存溢出

   申请内存时，没有足够的内存可以使用。

可见，内存泄漏和内存溢出的关系：内存泄漏的增多，最终导致内存溢出。

#### 泄漏的分类

经常发生：发生内存泄漏的代码会多次执行，每次执行，泄漏一块内存。

偶然发生：在某些特定情况下才会发生。

一次性：发生内存泄漏的方法只会执行一次。

隐式泄漏：一直占着内存不释放，知道执行结束；严格的说这个不算内存泄漏，因为最终释放掉了，但是如果执行时间特别长，也可能导致内存耗尽。

### Java中内存泄漏的8种情况

1. 静态集合类

   如HashMap、LinkedList等。如果这些容器是静态的,那么它们的生命周期与JVM程序一致，则容器中的对象在程序结束之前将不能被释放，从而造成内存泄漏。简而言之，长生命周期的对象持有短生命周期的对象引用，尽管短生命周期的对象不再使用，但是因为长生命周期对象持有它的引用而导致不能被回收。

~~~java
public class MemoryLeak {

    static List list = new ArrayList();

    public static void method(){
        Object o = new Object();
        list.add(o);
    }
}
~~~

2. 单例模式、

   和静态集合导致内存泄漏的原因类似，因为单例的静态特性，它的生命周期和JVM的生命周期一样长，所以如果单例对象如果持有外部对象的引用，那么这个外部对象也不会被回收，那么就会造成内存泄漏。

3. 内部类持有外部类

   如果一个外部类的实例对象的方法返回了一个内部类的实例对象。这个内部类对象被长期使用了， 即使那个外部类实例对象不再被使用，但由于内部类持有外部类的实例对象，这个外部类对象将不会被垃圾回收，这也会造成内存泄露。

4. 各种连接，如数据库连接、网络连接和IO连接等

   在对数据库进行操作的过程中，首先需要建立与数据库的连接，当不再使用时，需要调用close方法来释放与数据库的连接。只有连接被关闭后，垃圾回收器才会回收对应的对象。否则，如果在访问数据库的过程中，对Connection、Statement或ResultSet不显性地关闭，将会造成大量的对象无法被回收，从而引起内存泄漏。

~~~java
public static void main(String[] args) {
        try {
            Connection connection = null;
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("", "", "");
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery("");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            // 1.关闭结果集statement
            // 2.关闭声明的对象resultSet
            // 3.关闭连接 connection
        }
    }
~~~

5. 变量不合理的作用域

   一般而言，一个变量的定义的作用范围大于其使用范围，很有可能会造成内存泄漏。另一方面，如果没有及时地把对象设置为null，很有可能导致内存泄漏的发生。

~~~java
public class UsingRandom {

    private String msg;

    public void receiveMsg(){

        readFromNet();// 从网络中接受数据保存到msg中
        saveDB();// 把msg保存到数据库中
    }
}
~~~

如上面这个伪代码，通过readFromNet方法把接受的消息保存在变量msg中，然后调用saveDB方法把msg的内容保存到数据库中，此时msg已经就没用了，由于msg的生命周期与对象的生命周期相同，此时msg还不能回收，因此造成了内存泄漏。

实际上这个msg变量可以放在receiveMsg方法内部，当方法使用完，那么msg的生命周期也就结束，此时就可以回收了。还有一种方法，在使用完msg后，把msg设置为null，这样垃圾回收器也会回收msg的内存空间。

6. 改变哈希值

   当一个对象被存储进HashSet集合中以后，就不能修改这个对象中的那些参与计算哈希值的字段了。否则，对象修改后的哈希值与最初存储进HashSet集合中时的哈希值就不同了，在这种情况下， 即使在contains方法使用该对象的当前饮用作为的参数去HashSet集合中检索对象，也将返回找不到对象的结果，这也会导致无法从HashSet集合中单独删除当前对象，造成内存泄漏。

7. 缓存泄漏

   内存泄漏的另一个常见来源是缓存，一旦你把对象引用放入到缓存中，他就很容易遗忘，对于这个问题，可以使用WeakHashMap代表缓存，此种Map的特点是，当除了自身有对key的引用外，此key没有其他引用那么此map会自动丢弃此值

~~~java
public class MapTest {
    static Map wMap = new WeakHashMap();
    static Map map = new HashMap();
    public static void main(String[] args) {
        init();
        testWeakHashMap();
        testHashMap();
    }

    public static void init(){
        String ref1= new String("obejct1");
        String ref2 = new String("obejct2");
        String ref3 = new String ("obejct3");
        String ref4 = new String ("obejct4");
        wMap.put(ref1, "chaheObject1");
        wMap.put(ref2, "chaheObject2");
        map.put(ref3, "chaheObject3");
        map.put(ref4, "chaheObject4");
        System.out.println("String引用ref1，ref2，ref3，ref4 消失");

    }
    public static void testWeakHashMap(){

        System.out.println("WeakHashMap GC之前");
        for (Object o : wMap.entrySet()) {
            System.out.println(o);
        }
        try {
            System.gc();
            TimeUnit.SECONDS.sleep(20);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("WeakHashMap GC之后");
        for (Object o : wMap.entrySet()) {
            System.out.println(o);
        }
    }
    public static void testHashMap(){
        System.out.println("HashMap GC之前");
        for (Object o : map.entrySet()) {
            System.out.println(o);
        }
        try {
            System.gc();
            TimeUnit.SECONDS.sleep(20);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("HashMap GC之后");
        for (Object o : map.entrySet()) {
            System.out.println(o);
        }
    }

}
/** 结果
 String引用ref1，ref2，ref3，ref4 消失
 WeakHashMap GC之前
 obejct2=chaheObject2
 obejct1=chaheObject1
 WeakHashMap GC之后
 HashMap GC之前
 obejct4=chaheObject4
 obejct3=chaheObject3
 Disconnected from the target VM, address: '127.0.0.1:51628', transport: 'socket'
 HashMap GC之后
 obejct4=chaheObject4
 obejct3=chaheObject3
 **/
~~~

8. 监听器和回调

   内存泄漏另一个常见来源是监听器和其他回调，如果客户端在你实现的API中注册回调，却没有显示的取消，那么就会积聚。需要确保回调立即被当作垃圾回收的最佳方法是只保存他的若引用，例如将他们保存成为WeakHashMap中的键。

### 内存泄漏案例分析

~~~java

import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
~~~

上述程序并没有明显的错误，但是这段程序有一个内存泄漏，随着GC活动的增加，或者内存占用的不断增加，程序性能的降低就会表现出来，严重时可导致内存泄漏，但是这种失败情况相对较少。代码的主要问题在pop函数，下面通过这张图示展现
假设这个栈一直增长，增长后如下图所示

![内存泄漏案例图1](https://github.com/jackhusky/jvm/blob/main/docs/images/内存泄漏案例图1.jpg)

当进行大量的pop操作时，由于引用未进行置空，gc是不会释放的，如下图所示

![内存泄漏案例图2](https://github.com/jackhusky/jvm/blob/main/docs/images/内存泄漏案例图2.jpg)

解决方法：

~~~java
public Object pop() {
    if (size == 0)
    throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
~~~

一旦引用过期，清空这些引用，将引用置空。

![内存泄漏案例图3](https://github.com/jackhusky/jvm/blob/main/docs/images/内存泄漏案例图3.jpg)

## 支持使用OQL语言查询对象信息



## JProfiler 

### 基本概述

#### 介绍

在运行Java的时候有时候想测试运行时占用内存情况，这时候就需要使用测试工具查看了。JProfiler是一款Java应用性能诊断工具。功能强大， 但是收费。

#### 特点

- 使用方便、界面操作友好（简单且强大）
- 对被分析的应用影响小（提供模版）
- CPU，Thread，Memory分析功能尤其强大
- 支持对jdbc、nosql、jsp、servlet、socket等进行分析
- 支持多种模式（离线、在线）的分析
- 支持监控本地、远程的JVM
- 跨平台，拥有多种操作系统的安装版本

### 具体使用

#### 遥感监测Telemetries



#### 内存视图Live Memory

- 所有对象 All Objects

显示所有加载的类的列表和在堆上分配的实例数

- 记录对象 Record Objects

查看特定时间段对象的分配，并记录分配的调用堆栈。

- 分配访问树 Allocation Call Tree

显示一颗请求树或者方法、类包或对已选择类由带注释的分配信息的J2EE组件

- 分配热点 Allocation Hot Spots

显示一个列表，包括方法、类、包或分配已选择的J2EE组件。可以标注当前值并且显示差异值。对于每个热点都可以显示它的跟踪记录树

- 类追踪器 Class Tracker

类跟踪视图可以包含任意数量的图表，显示选定的类和包的实例与时间

分析：内存中的对象情况

1. 频繁创建Java对象：死循环、循环次数过多
2. 存在大的对象：读取文件时，byte[]应该边读边写
3. 存在内存泄漏

#### 堆遍历heap walker

**类 Classes**

显示所有类和它们的实例，可以点击具体的类“Used Selected Instance”实现进一步跟踪。

**分配 Allocations**

为所有记录对象显示分配树和分配热点。

**索引 References**

为单个对象和“显示到垃圾回收根目录的路径”提供索引图的显示功能。还能提供合并输入视图和输出视图的功能。

**时间 Time**

显示一个对已记录对象的解决时间的柱状图。

**检查 Inspections**

显示了一个数量的操作，将分析当前对象集在某种条件下的子集，实质是筛选的过程。

**图表 Graph**

需要在references视图和biggest视图手动添加对象到图表，它可以显示对象的传入和传出引用，能方便的找到垃圾收集器根源。

> 在工具栏点击“Go To Start”可以使堆内存重新计数，也就是回到初始状态。

#### cpu视图cpu views

JProfiler提供不同的方法来记录访问树以优化性能和细节。线程或者线程组以及线程状况可以被所有的视图选择。所有的视图都可以聚集到方法、类、包或J2EE组件等不同层上。

**访问树 Call Tree**

显示一个积累的自顶向下的树，树中包含所有在JVM中已记录的访问队列。JDBC，JMS和JNDI服务请求都被注释在请求树中。请求树可以根据Servlet和JSP对URL的不同需要进行拆分。

**热点 Hot Spots**

显示消耗时间最多的方法的列表。对每个热点都能够显示回溯树。该热点可以按照方法请求，JDBC，JMS和JNDI服务请求以及按照URL请求来进行计算。

**访问图 Call Graph**

显示一个从已选方法、类、包或J2EE组件开始的访问队列的图。

**方法统计 Method Statistcs**

显示一段时间内记录的方法的调用时间细节。

#### 线程视图threds

JProfiler通过对线程历史的监控判断其运行状态，并监控是否有线程阻塞产生，还能将一个线程所管理的方法以及树状形式呈现。

**线程历史 Thread History**

显示一个与线程活动和线程状态在一起的活动时间表。

**线程监控 Thread Monitor**

显示一个列表，包括所有的活动线程以及它们目前的活动状况。

**线程转储 Thread Dumps**

显示所有线程的堆栈跟踪。

线程分析主要关心三个方面：

1. web容器的线程最大数。比如：Tomcat的线程容量应该略大于最大并发数。
2. 线程阻塞。
3. 线程死锁。

#### 监视器&锁Monitors&locks

所有的线程持有锁的情况以及锁的信息。

- 死锁探测图表 Current Locking Graph：显示JVM中的当前死锁图表。
- 目前使用的检测器Current Monitors：显示目前使用的探测器并且包括它们的关联线程。
- 锁定历史图表 Locking History Graph：显示记录在JVM中的锁定历史。
- 历史监测记录：Monitor History：显示重大的等待事件和阻塞事件的历史记录。
- 监控器使用统计Monitor Usage Statistcs：显示分组监测，线程和监测类的统计监测数据。

## Arthas

### 基本概述 

Arthas是Alibaba开元的Java诊断工具，在线排查问题，无需重启；动态跟踪Java代码，实时监控JVM状态。

## Java Mission Control



## TProfiler

### 













