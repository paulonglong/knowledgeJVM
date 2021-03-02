# Class文件结构

## 概述

### 字节码文件的跨平台型

1. Java语言：跨平台的语言（write once，run anywhere）

- 当Java源代码成功编译成字节码后，如果想在不同的平台上面运行，则无需再次编译。
- 这个优势不再那么吸引人了。Python、PHP、Perl、Ruby等已有强大的解释器。
- 跨平台似乎已经快成为一门语言必选的特性。

2. Java虚拟机：跨语言的平台

​    Java虚拟机不和包括Java在内的任何语言绑定，它只与“Class文件”这种特定的二进制文件格式所关联。

![java虚拟机跨语言的平台](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/java虚拟机跨语言的平台.png)

3. 想要让一个Java程序正确地运行在JVM中，Java源码就必须要被编译为符合JVM规范的字节码。

- 前端编译器的主要任务就是负责将符合Java语法规范的Java代码转换为符合JVM规范的字节码。
- javac是一种能够将Java源码编译为字节码的前端编译器。
- javac编译器在将Java源码编译为一个有效的字节码文件过程中经历了4个步骤，分别是**词法解析、语法解析、语义解析以及生成字节码**。

![源码编译](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/源码编译.png)

Oracle的JDK软件包括两部分内容：

- 一部分是将Java源代码编译成Java虚拟机的指令集的编译器。
- 另一部分是用于实现Java虚拟机的运行时环境。

### Java的前端编译器

javac是一种能够将Java源码编译为字节码的前端编译器。

除了javac之外，还有Eclipse中的ECJ（Eclipse Compiler for Java）编译器。和javac的全量编译不同，ECJ是一种增量式编译器。

## 虚拟机的基石：Class文件

- 字节码文件里是什么？

源代码经过编译器编译后便会生成一个字节码文件，字节码是一种二进制的类文件，它的内容是JVM的指令，而不像C、C++经由编译器直接生成机器码。

- 什么是字节码指令？

Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的操作码（opcode）以及跟随其后的零至多个代表此操作所需参数的操作数（operand）所构成。虚拟机中许多指令并不包含操作数，只有一个操作码。

- 如何解读供虚拟机解释执行的二进制字节码？

方式一：一个一个二进制的看。使用Notepad++，安装一个HEX-Editor插件，或者使用Binary Viewer。

方式二：使用javap指令：jdk自带的反解析工具。

方式三：使用IDEA插件：jclasslib 或者 jclasslib bytecode viewer客户端工具。

### Class类的本质

任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，Class文件实际上它并不一定以磁盘文件的形式存在。Class文件是一组以8位字节为基础单位的**二进制流**。

### Class文件格式

它的结构没有任何分隔符号。其中的数据项，无论是字节顺序还是数量，都是被严格限定的，哪个字节代表什么含义，长度是多少，先后顺序如何，都不允许改变。

Class文件格式采用一种类似于C语言结构体的方式进行数据存储，这种结构中只有两种数据类型：无符号数和表。

- 无符号数属于基本数据类型，以u1、u2、u4、u8来表别代表1个字节、2个字节、4个字节、8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或按照UTF-8编码构成字符串值。
- 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯地以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。由于表没有固定长度，所以通常会在其前面加上个数说明。

## Class文件结构

Class文件结构并不是一成不变的，随着Java虚拟机的不断发展，总是不可避免地会对Class文件结构做出一些调整，但是其基本结构和框架是非常稳定的。Class文件的总体结构如下：

- 魔数
- Class文件版本
- 常量池
- 访问标志
- 类索引，父类索引，接口索引集合
- 字段表集合
- 方法表集合
- 属性表集合

| 类型           | 名称                | 说明                   | 长度    | 数量                  |
| -------------- | ------------------- | ---------------------- | ------- | --------------------- |
| u4             | magic               | 魔数,识别Class文件格式 | 4个字节 | 1                     |
| u2             | minor_version       | 副版本号(小版本)       | 2个字节 | 1                     |
| u2             | major_version       | 主版本号(大版本)       | 2个字节 | 1                     |
| u2             | constant_pool_count | 常量池计数器           | 2个字节 | 1                     |
| cp_info        | constant_pool       | 常量池表               | n个字节 | constant_pool_count-1 |
| u2             | access_flags        | 访问标识               | 2个字节 | 1                     |
| u2             | this_class          | 类索引                 | 2个字节 | 1                     |
| u2             | super_class         | 父类索引               | 2个字节 | 1                     |
| u2             | interfaces_count    | 接口计数器             | 2个字节 | 1                     |
| u2             | interfaces          | 接口索引集合           | 2个字节 | interfaces_count      |
| u2             | fields_count        | 字段计数器             | 2个字节 | 1                     |
| field_info     | fields              | 字段表                 | n个字节 | fields_count          |
| u2             | methods_count       | 方法计数器             | 2个字节 | 1                     |
| method_info    | methods             | 方法表                 | n个字节 | methods_count         |
| u2             | attributes_count    | 属性计数器             | 2个字节 | 1                     |
| attribute_info | attributes          | 属性表                 | n个字节 | attributes_count      |

### 魔数

- 每个Class文件开头的4个字节的无符号整数成为魔数。
- 它的唯一作用是确定这个文件是否为一个能被虚拟机接收的有效合法的Class文件。即：魔数是Class文件的标识符。
- 魔数值固定为0xCAFEBABE。
- 使用魔数而不是扩展名来进行识别主要是基于安全方面的考虑，因为文件扩展名可以随意改动。

### Class文件版本号

紧接着魔数的4个字节存储的是Class文件的版本号。第5个和第6个字节所代表的含义就是编译的副版本号minor_version，而第7个和第8个字节就是编译的主版本号major_version。

它们共同构成了class文件的格式版本号。譬如某个Class文件的主版本号是M，副版本号是m，那么这个Class文件的格式版本号就确定为M.m。

版本号和Java比哪一期的对应关系如下表：

| 主版本（十进制） | 副版本（十进制） | 编译器版本 |
| ---------------- | ---------------- | ---------- |
| 45               | 3                | 1.1        |
| 46               | 0                | 1.2        |
| 47               | 0                | 1.3        |
| 48               | 0                | 1.4        |
| 49               | 0                | 1.5        |
| 50               | 0                | 1.6        |
| 51               | 0                | 1.7        |
| 52               | 0                | 1.8        |
| 53               | 0                | 1.9        |
| 54               | 0                | 1.10       |
| 55               | 0                | 1.11       |

### 常量池

在版本号之后，紧跟着的是常量池的数量，以及若干个常量池表项。

常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的无符号数，代表常量池容量计数值。与Java中语言习惯不一样的是，这个容量计数是从1而不是0开始的。

常量池表项中，用于存放编译时期生成的各种字面量和符号引用，这部分内容在类加载后进入方法区的运行时常量池中存放。

#### constant_pool_count（常量池计数器）

常量池容量计数值（u2类型）：从1开始，表示常量池中有多少项常量。即  constant_pool_count=1表示常量池中有0个常量项。

若值为0x0016，也就是22。需要注意的是，这实际上只有21项常量。索引范围是1-21。为什么呢？

这里的常量池是从1开始，因为它把第0项常量空出来了。这是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况可用索引值0来标示。

#### constant_pool[]（常量池）

constant_pool是一种表结构，以1~constant_pool_count-1为索引。标明后面有多少个常量项。

常量池主要存放两大类常量：字面量（Literal）和符号引用（Symbolic Reference）

它包含了class文件结构及其子结构中引用的所有字符串常量、类或接口名、字段名和其它常量。常量池中的每一项都具备相同的特征。第1个字节作为类型标记，用于确定该项的格式，这个字节成为tag byte（标记字节、标签字节）。

| 类型                             | 标志(或标识) | 描述                   |
| -------------------------------- | ------------ | ---------------------- |
| CONSTANT_utf8_info               | 1            | UTF-8编码的字符串      |
| CONSTANT_Integer_info            | 3            | 整型字面量             |
| CONSTANT_Float_info              | 4            | 浮点型字面量           |
| CONSTANT_Long_info               | 5            | 长整型字面量           |
| CONSTANT_Double_info             | 6            | 双精度浮点型字面量     |
| CONSTANT_Class_info              | 7            | 类或接口的符号引用     |
| CONSTANT_String_info             | 8            | 字符串类型字面量       |
| CONSTANT_Fieldref_info           | 9            | 字段的符号引用         |
| CONSTANT_Methodref_info          | 10           | 类中方法的符号引用     |
| CONSTANT_InterfaceMethodref_info | 11           | 接口中方法的符号引用   |
| CONSTANT_NameAndType_info        | 12           | 字段或方法的符号引用   |
| CONSTANT_MethodHandle_info       | 15           | 表示方法句柄           |
| CONSTANT_MethodType_info         | 16           | 标志方法类型           |
| CONSTANT_InvokeDynamic_info      | 18           | 表示一个动态方法调用点 |

#### 字面量和符号引用

| 常量     | 具体的常量          |
| -------- | ------------------- |
| 字面量   | 文本字符串          |
|          | 声明为final的常量值 |
| 符号引用 | 类和接口的全限定名  |
|          | 字段的名称和描述符  |
|          | 方法的名称和描述符  |

字面量：

```java
String str = "abc";
final int number = 10;
```

全限定名：

com/demo/test/Demo这个就是类的全限定名。为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“;”表示全限定名结束。

简单名称：

就是指没有类型和参数修饰的方法或者字段名称，比如类的add()方法和num字段的简单名称就是add和num。

描述符：

作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。

| 标志符 | 含义                                                 |
| ------ | ---------------------------------------------------- |
| B      | 基本数据类型byte                                     |
| C      | 基本数据类型char                                     |
| D      | 基本数据类型double                                   |
| F      | 基本数据类型float                                    |
| I      | 基本数据类型int                                      |
| J      | 基本数据类型long                                     |
| S      | 基本数据类型short                                    |
| Z      | 基本数据类型boolean                                  |
| V      | 代表void类型                                         |
| L      | 对象类型，比如：`Ljava/lang/Object;`                 |
| [      | 数组类型，代表一维数组。比如：`double[][][] is [[[D` |

虚拟机在加载Class文件时才会进行动态链接，也就是说，Class文件中不会保存各个方法和字段的最终内存布局信息，因此，这些字段和方法的符号引用不经过转换是无法直接被虚拟机使用的。虚拟机在运行时，需要从常量池中获得对应的符号引用，再在类加载过程中的解析阶段将其替换为直接引用，并翻译到具体的内存地址中。

- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到了内存中。
- 直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那说明引用的目标必定存在于内存中了。

![常量类型和结构细节1](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/常量类型和结构细节1.png)

![常量类型和结构细节2](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/常量类型和结构细节2.png)

### 访问标识（access_flag、访问标志、访问标记）

在常量池后，紧跟着访问标记。该标记使用两个字节表示，用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为`public`类型；是否定义为`abstract`类型；如果是类的话，是否被声明为`final`等。

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 标志为public类型                                             |
| ACC_FINAL      | 0x0010 | 标志被声明为final，只有类可以设置                            |
| ACC_SUPER      | 0x0020 | 标志允许使用invokespecial字节码指令的新语义，JDK1.0.2之后编译出来的类的这个标志默认为真。（使用增强的方法调用父类方法） |
| ACC_INTERFACE  | 0x0200 | 标志这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，此标志值为真，其他类型为假 |
| ACC_SYNTHETIC  | 0x1000 | 标志此类并非由用户代码产生（即：由编译器产生的类，没有源码对应） |
| ACC_ANNOTATION | 0x2000 | 标志这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标志这是一个枚举                                             |

- 类的访问权限通常为ACC_开头的常量。
- 每一种类型的表示都是通过设置访问标记的32位中的特定位来实现的。比如：若是`public final`的类，则该标记为ACC_PUBLIC |ACC_FINAL。
- 使用ACC_SUPER可以让类更准确地定位到父类的方法super.method()，现代编译器都会设置并且使用这个标记。

### 类索引，父类索引，接口索引集合

在访问标识后，会指定该类的类别、父类类别以及实现的接口

| 长度 | 含义                         |
| ---- | ---------------------------- |
| u2   | this_class                   |
| u2   | super_class                  |
| u2   | interfaces_count             |
| u2   | interfaces[interfaces_count] |

### 字段表集合

fields

- 用于描述接口或类中声明的变量。字段（field）包括类级变量以及实例变量，但是不包括方法内部、代码块内部声明的局部变量。
- 字段叫什么名字、字段被定义什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。
- 它指向常量池索引集合，描述了每个字段的完整信息。比如字段的标识符、访问修饰符（`public`、`private`或`protected`）、是类变量还是实例变量（`static`修饰符）、是否是常量（`final`修饰符）等。

注意事项：

- 字段表集合中不会列出从父类或者实现的接口中继承来的字段，但有可能列出原本Java代码中不存在的字段。比如内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段。
- 在Java语言中字段是无法重载的，两个字段的数据类型、修饰符不管是否相同，都必须使用不一样的名称，但是对于字节码来讲，如果两个字段的描述符不一致，那字段重名就是合法的。

#### fileds_count（字段计数器）

fileds_count的值表示当前class文件fileds表的成员个数。使用两个字节来表示。

fileds表中每个成员都是filed_info结构，用于表示该类或接口所声明的所有类字段或者实例字段，不包括方法内部声明的变量，也不包括从父类或父接口继承的字段。

#### fileds[]（字段表）

- fileds表中的每个成员都必须是一个fileds_info结构的数据项，用于表示当前类或接口中某个字段的完整描述。
- 一个字段的信息包括如下这些信息。这些信息中，各个修饰符都是布尔值，要么有，要么没有。
  - 作用域（`public`、`private`、`protected`修饰符）
  - 是实例变量还是类变量（`static`修饰符）
  - 可变性（`final`）
  - 并发可见性（`volatile`修饰符，是否强制从主内存读写）
  - 可否序列化（`tracsient`修饰符）
  - 字段数据类型（基本数据类型、对象、数组）
  - 字段名称
- 字段表结构

| 类型           | 名称             | 含义       | 数量             |
| -------------- | ---------------- | ---------- | ---------------- |
| u2             | access_flags     | 访问标志   | 1                |
| u2             | name_index       | 字段名索引 | 1                |
| u2             | descriptor_index | 描述符索引 | 1                |
| u2             | attributes_count | 属性计数器 | 1                |
| attribute_info | attributes       | 属性集合   | attributes_count |

##### 字段表访问标识

| 标志名称      | 标志值 | 含义                       |
| ------------- | ------ | -------------------------- |
| ACC_PUBLIC    | 0x0001 | 字段是否为public           |
| ACC_PRIVATE   | 0x0002 | 字段是否为private          |
| ACC_PROTECTED | 0x0004 | 字段是否为protected        |
| ACC_STATIC    | 0x0008 | 字段是否为static           |
| ACC_FINAL     | 0x0010 | 字段是否为final            |
| ACC_VOLATILE  | 0x0040 | 字段是否为volatile         |
| ACC_TRANSTENT | 0x0080 | 字段是否为transient        |
| ACC_SYNCHETIC | 0x1000 | 字段是否为由编译器自动产生 |
| ACC_ENUM      | 0x4000 | 字段是否为enum             |

##### 字段名索引

根据字段名索引的值，查询常量池中的指定索引项即可。

##### 描述符索引

描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型及代表无返回值的void类型都用一个大写字符来表示，而对象则用字符L加对象的全限定名来表示

| 字符          | 类型      | 含义                    |
| ------------- | --------- | ----------------------- |
| B             | byte      | 与符号字节型数          |
| C             | char      | Unicode字符，UTF-16编码 |
| D             | double    | 双精度浮点数            |
| F             | float     | 单精度浮点数            |
| I             | int       | 整型数                  |
| J             | Long      | 长整形                  |
| S             | short     | 有符号短整数            |
| Z             | boolean   | 布尔值true/false        |
| L Classname； | reference | 一个名为Classname的实例 |
| [             | reference | 一个一维数组            |

##### 属性表集合	

一个字段还可能拥有一些属性，用于存储更多的额外信息。比如初始化值、一些注释信息等。属性个数存放在attribute_count中，属性具体内容存放在attribute数组中。

以常量属性为例，结构为：

```java
ConstantValue_attribute{

	u2 attribute_name_index;

	u4 attribute_length;	

	u2 constantvalue_index;

}
```

对于常量属性而言，attribute_length值恒为2。

### 方法表集合

methods：指向常量池索引集合，它完整描述了每个方法的签名。

- 字节码文件中，每个method_info项都对应着一个类或者接口中的方法信息。比如方法的访问修饰符（public、private、protected），方法的返回值类型以及方法的参数信息等。
- 如果这个方法不是抽象的或者不是native的，那么字节码中会体现出来。
- 一方面，methods表只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法。另一方面，mehods表有可能会出现由编译器自动添加的方法，最典型的便是编译器产生的方法信息（比如类初始化方法`<clinit>()`和实例初始化方法`<init>()`）。

注意事项：

如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法共存于同一个class文件中。

#### methods_count（方法计数器）

methods_count的值表示当前class文件methods表的成员个数。使用两个字节来表示。

methods表每个成员都是一个method_info结构。

#### methods[]（方法表）

- methods表中的每个成员都必须是一个method_info结构，用于表示当前类或接口中某个方法的完整描述。如果某个method_info结构的access_flags项既没有设置ACC_NATIVE标识也没有设置ACC_ABSTRACT标识，那么该结构中也应包含实现这个方法所用的Java虚拟机指令。
- method_info结构可以表示类和接口中定义的所有方法，包括实例方法，类方法，实例初始化方法和类或接口初始化方法。

- 方法表结构实际和字段表是一样的

| 类型           | 名称             | 含义       | 数量             |
| -------------- | ---------------- | ---------- | ---------------- |
| u2             | access_flags     | 访问标志   | 1                |
| u2             | name_index       | 方法名索引 | 1                |
| u2             | descriptor_index | 描述符索引 | 1                |
| u2             | attributes_count | 属性计数器 | 1                |
| attribute_info | attributes       | 属性集合   | attributes_count |

##### 方法表访问标志

跟字段表一样，方法表也有访问标志，而且他们的标志有部分相同，部分则不同

### 属性表集合

方法表集合后的属性表集合，指的是class文件所携带的辅助信息，比如该class文件的源文件的名称。以及任何带有RetentionPolicy.CLASS或者RetentionPolicy.RUNTIME的注解。这类信息通常被用于Java虚拟机的验证和运行，以及Java程序的调试，一般无需深入了解。

此外，字段表、方法表都可以有自己的属性表。用于描述某些场景专有的信息。

#### attributes_count（属性计数器）

attributes_count的值表示当前class文件属性表的成员个数，属性表中每一项都是一个attribute_info结构。

#### attributes[]（属性表）

属性表的每个项的值必须是attribute_info结构。属性表的结构比较灵活，各种不同的属性只要满足以下结构。

##### 属性的通用格式

| 类型 | 名称                 | 数量             | 含义       |
| ---- | -------------------- | ---------------- | ---------- |
| u2   | attribute_name_index | 1                | 属性名索引 |
| u4   | attribute_length     | 1                | 属性长度   |
| u1   | info                 | attribute_length | 属性表     |

只需说明属性的名称以及占用位数的长度即可，属性表具体的结构可以去自定义。

##### 属性类型

属性表实际上可以有很多类型，上面看到的Code属性是其中一种，[Java8](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7)定义了23种属性。

![字节码解析](https://github.com/paulonglong/knowledgeJVM/blob/master/docs/images/字节码解析.png)

## 使用javap指令解析Class文件

### javac -g操作

解析字节码文件得到的信息中，有些信息（如局部变量表、指令和代码行偏移量映射表、常量池中方法的参数名称等）需要在使用javac编译成class文件时，指定参数才能输出。

如果直接使用javac xx.java不会生成对应的局部变量表等信息，若使用了javac -g xx.java就可以生成所有相关信息了。默认情况下，IDEA或eclipse在编译时会生成局部变量表、指令和代码行偏移量映射表等信息。

### javap主要参数

```shell
# 仅显示公共类和成员
javap -public xxx.class
# 显示受保护的/公共类和成员
javap -protected xxx.class
# 显示所有类和成员
javap -p/-private xxx.class
# 显示程序包/受保护的/公共类和成员（默认）
javap -package xxx.class
# 显示正在处理处理的类的系统信息(路径，大小，日期，MD5散列，源文件名)
javap -sysinfo xxx.class
# 显示静态最终常量
javap -constants xxx.class

# 输出内部类型签名
javap -s xxx.class
# 输出行号和本地变量表
javap -l xxx.class
# 对代码进行反汇编
javap -c xxx.class
# 输出附加信息(包括行号、本地变量表、反汇编等详细信息)
javap -v/-verbose xxx.class
```
