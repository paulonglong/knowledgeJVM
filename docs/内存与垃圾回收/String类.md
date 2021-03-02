# String类运行时管理

## String的基本特性

- String：代表不可变的字符序列。简称：不可变性。
- 通过字面量的方式给一个字符串赋值，此时的字符串值声明在字符串常量池中。
- 字符串常量池中不会存储相同内容的字符串的。
- String的String Pool是一个固定大小的`HashTable`，如果字符串非常多，就会造成Hash冲突严重，从而导致链表会很长，链表长了以后直接会造成的影响就是当调用`String.intern()`时性能会大幅下降。
- 使用`-XX:StringTableSize`可设置`StringTable`的长度。
- JDK6中`StringTable`是固定的，就是**1009**的长度，如果常量池中的字符串过多就会导致效率下降很快。`StringTableSize`设置没有要求。
- JDK7中，`StringTable`的长度默认是**60013**。
- JDK8开始，设置`StringTable`长度的话，1009是可设置的最小值。

## String的内存分配

- Java 6及以前，字符串常量池存放在永久代。
- Java 7中Oracle的工程师对字符串池的逻辑做了很大的改变，将字符串常量池的位置调整到Java堆中。
- Java 8元空间，字符串常量在堆。

问题：`StringTable`为什么调整？

1. PermSize默认比较小
2. 永久代垃圾回收频率低

## String的基本操作

## 字符串拼接操作

1. 常量与常量的拼接结果在常量池，原理是编译器优化。
2. 常量池中不会存在相同内容的常量。
3. 只要其中有一个是变量，结果就在堆中，变量拼接的原理是`StringBuilder`。
4. 如果拼接的结果调用`intern()`方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

```java
public class StringTest {

    @Test
    public void test1(){
        String s1 = "a" + "b" + "c"; //等同于"abc"，进行反编译也是"abc"
        String s2 = "abc"; // "abc"放在字符串常量池中，将地址付给s2

        System.out.println(s1 == s2); //true
        System.out.println(s1.equals(s2)); //true
    }

    @Test
    public void test2(){
        String s1 = "javaEE";
        String s2 = "hadoop";

        String s3 = "javaEEhadoop";
        String s4 = "javaEE" + "hadoop"; // 编译器优化 "javaEEhadoop"
        // 如果拼接符号前后出现了变量，相当于在堆空间中new String()，具体内容为拼接的结果javaEEhadoop
        String s5 = s1 + "hadoop";
        String s6 = "javaEE" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4); // true
        System.out.println(s3 == s5); // false
        System.out.println(s3 == s6); // false
        System.out.println(s3 == s7); // false
        System.out.println(s5 == s7); // false
        System.out.println(s6 == s7); // false
        // 判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址
        // 如果不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回此对象的地址
        String s8 = s6.intern();
        System.out.println(s3 == s8); // true
    }

    @Test
    public void test3(){
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        /**
         * StringBuilder s = new StringBuilder();
         * s.append("a");
         * s.append("b");
         * s.toString()  --> 约等于 new String("ab");
         */
        String s4 = s1 + s2;
        System.out.println(s3 == s4); // false
    }

    /**
     * 1、拼接符号左右两边都是字符串常量或者常量引用，使用编译期优化，不是StringBuilder的方式
     * 2、针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用final的时候建议使用上
     */
    @Test
    public void test4(){
        final String s1 = "a";
        final String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;
        System.out.println(s3 == s4); // true
    }
}
```

实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel，使用构造器

```java
StringBuilder s = new StringBuilder(highLevl);
```

## intern()的使用

如果不是用双引号声明的`String`对象，可以使用`String`提供的`intern()`方法，`intern()`方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

```java
String s = new String("abc").intern();
```

调用`String.intern`方法，那么返回结果指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。

```java
("a" + "b" + "c").intern() == "abc"
```

确保字符串在内存中只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串常量池。

题目：`new String(“ab”)`会创建几个对象？两个

一个对象是：`new`关键字在堆空间创建的。

一个对象是：字符串常量池中的对象“ab”。字节码指令：`ldc`

`new String(“a”) + new String(“b”)`呢？

1、`new StringBuilder()`

2、`new String(“a”)`

3、常量池中的“a”

4、`new String(“b”)`

5、常量池中的“b”

深入剖析：`StringBuilder`的`toString()`

6、`new String(“ab”)`，`toString()`的调用，在字符串常量池中，没有生成“ab”

```java
public static void main(String[] args) {
    String s = new String("1");
    // 调用此方法之前，字符串常量池中已经存在了"1"
    s.intern();
    String s2 = "1";
    System.out.println(s == s2); // jdk6:false jdk7/8:false

    // s3变量记录的地址为new String("11")
    String s3 = new String("1") + new String("1");
    // 执行完上一行代码后，字符串常量池中，是否存在”11“呢？不存在
    // 在字符串常量池中生成"11" 如何理解
    // jdk6:创建一个新的对象"11",也就有新的地址
    // jdk7:此时常量池没有创建"11"，而是创建一个指向堆空间new String("11")的地址
    s3.intern();
    // s4变量记录的地址:使用的是上一行代码执行时，在常量池中生成的"11"的地址
    String s4 = "11";
    System.out.println(s3 == s4); // jdk6:false jdk7/8:true
}
```

JDK6：

![jdk6intern()](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/jdk6intern().png)

JDK7/8：

![jdk7intern()](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/jdk7intern().png)

总结intern()的使用

- JDK6中，将这个字符串对象尝试放入常量池。
  - 如果常量池中有，则并不会放入。返回已有的常量池中的对象的地址。
  - 如果没有，会把此对象复制一份(new)，放入常量池，并返回常量池中的对象地址。
- JDK7起，将这个字符串对象尝试放入常量池。
  - 如果常量池中有，则并不会放入。返回已有的常量池中的对象的地址。
  - 如果没有，会把此对象的引用地址复制一份，放入常量池，并返回常量池中的引用地址。

练习：

![练习1intern](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/练习1intern.png)

![练习1jdk7intern](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/练习1jdk7intern.png)

![练习1jdk8intern变形](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/练习1jdk8intern变形.png)

```java
    @Test
    public void test5(){
		//String s1 = new String("ab"); //会在字符串常量池中生成"ab" false
        String s1 = new String("a") + new String("b"); //不会在字符串常量池中生成"ab" true
        s1.intern();
        String s2 = "ab";
        System.out.println(s1 == s2);
    }
```

效率测试：对于程序中大量存在的字符串，尤其存在很多重复的字符串，使用intern()可以节省内存空间。

## StringTable的垃圾回收

`-XX:+PrintStringTableStatistics`:打印字符串常量池的统计信息

## G1中的String去重操作

Java堆中存活的数据集合差不多25%是`String`对象，这里面差不多一半`String`对象是重复的。`str1.equals(str2)=true`。堆上存在重复的`String`对象必然是一种内存的浪费。

实现：

- 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的`String`对象。

- 使用hashtable记录所有的被`String`对象使用的不重复的`char`数组。去重的时候会查这个`hashtable`，来看堆上是否存在一个一样的`char`数组。

- 如果存在`String`对象会被调整引用那个数组，释放原来的数组的引用，最终会被垃圾收集器回收掉。
- 不存在，`char`数组会被插入到`hashtable`，这样以后就可以共享这个数组了。

- 命令行选项
  - `UseStringDeduplication`：开启String去重，默认不开启。
  - `PrintStringDeduplicationStatistics`：打印详细的去重统计信息。
  - `StringDeduplicationAgeThreshold`：达到这个年龄的`String`对象被认为是去重的候选对象。
