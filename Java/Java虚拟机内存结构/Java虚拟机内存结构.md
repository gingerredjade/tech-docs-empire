# Java虚拟机内存结构

Java设计之初是用C++设计的，

Java虚拟机（Java Virtual Machine, JVM），一种能够运行Java字节码的虚拟机。作为一种编程语言的虚拟机，实际上不只是专用于Java语言，只要生成的编译文件匹配JVM对加载编译文件格式要求，任何语言都可以由JVM编译运行。比如kotlin、Scala等。

## JVM简介

Sun hotspot，微软vitural machine，阿里巴巴出品的虚拟机，这些虚拟机都是基于Java虚拟机规范的一种实现。不同的虚拟机可以解决各种不同业务场景。

JVM偏向于软件层面的虚拟机，Java虚拟机虽然没有一整套的硬件体系，但是**Java虚拟机有自己独特的指令集、可执行文件结构**，其中可执行的文件结构指的就是class字节码文件。

## JVM基本结构

JVM由三个主要的子系统构成

- 类加载子系统（掌握类加载器、类加载机制[双亲委派、全盘负责委托，怎么样打破双亲委派？]）
- 运行时数据区（内存结构）
- 执行引擎



加载阶段，通过类加载器加载类。以后通过类加载器和它的全限定名来确定一个类。不同的类加载器去加载相同的class文件属于不同的类。

![Java Virtual Machine architecture diagram ](https://javatutorial.net/wp-content/uploads/2017/10/jvm-architecture-992x1024.png)

（上图源自https://javatutorial.net/jvm-explained）

.class文件通过类加载子系统（双亲委派模型、全盘负责委托机制等）加载进来之后，一个class文件可以被分成多个部分加载到运行时数据区，然后可以通过执行引擎来执行运行时数据区的数据。

一个class文件可以被分解为多个部分加载，如：

- class文件模板文件本身可以放在方法区；
- 创建出来的实例对象可以放在堆里；
- 栈：;
- 程序计数器：；

- 本地方法栈：；

### 类加载机制

#### 类的生命周期

类加载包含了以下阶段：

![image-20191217095050715](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217095050715.png)

![image-20191217144243466](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217144243466.png)

​        JIT：运行时编译。



##### 1.加载

将.class文件从磁盘读到内存。

##### 2.连接

###### 2.1验证

验证字节码文件的正确性，包括元数据格式验证、模数验证等。

###### 2.2准备

给类的静态变量分配内存，并赋予默认值。

###### 2.3解析

类装载器装入类所引用的其他所有类。（静态链接）

##### 3.初始化

为类的静态变量赋予正确的初始值，上述的准备阶段为静态变量赋予的是虚拟机默认的初始值，此处赋予的才是程序编写者为变量分配的真正的初始值，执行静态代码块。

##### 4.使用

##### 5.卸载



#### 类加载器的种类

##### 启动类加载器（Bootstrap ClassLoader）

负责加载JRE的核心类库，如JRE目标下的rt.jar,charset.jar等。

##### 扩展类加载器（Extension ClassLoader）

负责加载JRE扩展目录ext中jar类包。

##### 系统类加载器（Application ClassLoader）

负责加载ClassPath路径下的类包。

##### 用户自定义类加载器（User ClassLoader）

负责加载用户自定义路径下的类包

![image-20191217145152663](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217145152663.png)



#### 类加载机制

##### 全盘负责委托机制

当一个ClassLoader加载一个类的时候，除非显式的使用另一个ClassLoader，该类所依赖和引用的类也由这个ClassLoader载入。

##### 双亲委派机制

指先委托父类加载器寻找目标类，在找不到的情况下在自己的路径中查找并载入目标类。

##### 双亲委派模式的优势

- 沙箱安全机制：比如自己写的String.class类不会被加载，这样可以防止核心库被随意篡改。
- 避免类的重复加载：当父ClassLoader已经加载了该类的时候，就不需要子ClassLoader再加载一次。



### 运行时数据区（内存结构）

堆、方法区：属于线程共享的。

虚拟机栈、本地方法栈、程序计数器：是本地私有的。

![](C:%5CUsers%5Cginge%5CDesktop%5Con6nhdrdag.png)

#### 1.方法区（Method Area）

类的所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在这里定义。简单来说，所有定义的方法的信息都保存在该区域，静态变量+常量+类信息即类的模板文件（构造方法/接口定义）+运行时常量池都存在方法区中，虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap(非堆，也叫堆外内存)，目的应该是为了和Java的堆区分开。

(hotspot)  （Java一直用的）、BEA jrokit（最快的）

1.6时：将方法区称为永久代或者持久代

1.7时：逐步移除，但是还用的是永久代

1.8时：完全移除，使用元空间（hotspot、jrokit做了融合）



- young GC minorGC：新生代产生的GC，执行速度快，回收内存小。

- FullGC MajorGC：老年代产生的GC，STW问题（停止时间）。

  - 吞吐量
  - 停顿时间

  这两个指标都和STW有关。

  FullGC在元空间、老年代放不下东西时可能会发生。一般发生FullGC都会伴随着一次清GC也就是回收所有的堆区域。因为元空间在逻辑上是属于堆的，所以一般元空间也会回收掉。所以FullGC会有一个执行时间过长的特点。

- mixedGC



#### 2.堆（Heap）

- **堆是Java存储的一个单位**；

虚拟机启动时自动分配创建，用于存放对象的实例，几乎所有对象都在堆上分配内存，当对象无法在该空间申请到内存时将抛出OutOfMemoryError异常。同时也是垃圾收集器管理的主要区域。

![image-20191217101458208](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217101458208.png)

![image-20191217102934169](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217102934169.png)

From<-->To这块区域又称作Survivor区。

堆和元空间都是我们能够管到的两个空间，堆其实就是一块不规则、不连续的内存空间。

Eden：初生对象，如果比较小的话可以放到Eden区。因为JVM中还有分配担保机制，新生代只占了堆的1/3内存，老年代占2/3，新生代中Eden和Survivor区占比是8:1:1。Eden区比较大的对象就直接放老年代。Eden区比较小，放满了就需要用垃圾收集器进行清理回收无用对象，剩下的存活对象转移到Survivor区（幸存者区）。

Survivor区分为From、To两块区域。一旦对象进入了Survivor区，头上就顶着一个年龄，这个年龄会伴随它转移到老年代为止。Eden区先转移到From Survivor，如果From Survivor也满了，会对From区域进行垃圾回收，之后将其中对象转移到To Survivor区域，转移完后To Survivor区域变成了From Survivor区域。这样去是为了设置年龄、方便实现赋值算法。对象从From Survivor区域转移到To Survivor区域这一过程对象的年龄加1，当加到了15（JVM参数默认值）的时候，就转移到老年代。

堆内存时不连续的，比如说From Survivor区有空闲内存，但是不连续，当几个连续的存活对象转移到From Survivor来的时候存储不下了，就会对From Survivor区域进行垃圾回收将其中对象转移道To Survivor区域，这样到To Survivor区域中被转移对象是排好序的，To Survivor区域剩下的就是一块连续的内存空间（注意此时To Survivor区域变成了From Survivor区域），可以方便我们往里面放对象。

为什么是默认15呢？这个涉及到JVM规范。

在HotSpot虚拟机中，每一个对象都有对象的布局，C++在设计Java时有Java的设计文件oop。对象布局可看成是对象的组成部分，对象在内存中存储的布局可以分为3块：对象头、实例数据和对齐填充，每个部分都有组成设计。

对象头：markwork、class。HotSpot底层通过markOop实现Mark Word，具体实现位于markOop.hpp文件。里面age:4, 四个零0000最大是15.

实例数据：实例数据是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

对齐填充：对齐填充不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数，因此，但对象实例数据部分没有对齐时，就需要对齐填充来补全。

可参考：https://www.baidu.com/link?url=CzEz_9JrOTgaFFUAheFyz7g2aH8BraPx_CT1j08r19HXEM_iKd88PSDB_ee-kcwx&wd=&eqid=89c85d1b0002e11b000000025df841c8



老年代终究也会满，老年代也会发生GC，但是这个GC和新生代GC（Young GC）不一样。

![image-20191217110656671](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217110656671.png)

垃圾收集器有很多种，可以组合使用。G1垃圾收集器是个能够处理数据级别比较高的垃圾收集器。JDK1.8默认用的是Parallel Scavenge。一般是组合使用，只有G1是两个区域都回收的。不同的垃圾回收器的算法不同，回收的是不同的代的。

JVM在做FullGC时会回收所有区域执行比较慢，并且Java会先停掉用户线程专门去执行垃圾收集线程，把要标记的对象回收掉，回收完后再去执行用户线程。这是产生STW的原因。

判断何时进行垃圾回收：看JVM使用的是哪种垃圾回收方式：引用计数法、可达性分析法。这两种都是当对象没有任何引用的时候就可以被回收了。

平时讲的JVM调优主要指两块：方法区、堆。

调方法区的时候不多，调过的一个是eureka，因为它里面加载的类过多，会造成方法区创建非常多的class模板文件，class模板文件过多的话元空间很快就满了，并且元空间里有个动态扩容机制：比如给元空间分配了128M内存，元空间实际上只有21M可以使用，这是为了防止内存浪费。当到达21M水平线的时候会执行一次FullGC，根据GC日志判断回收情况，如果回收完之后发现里面的数据还是21M左右，会把水平线上涨；如果回收完之后变成了0KB，水平线就下降，确保内存不浪费。如果项目在启动的时候发现元空间发生了几次FullGC没关系，可以把元空间调大一些。



##### 2.1新生代（Young Generation）

类出生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。

新生代分为两部分：伊甸区（Eden Space）和幸存者区（Survivor Space），所有的类都是在伊甸区被new出来的。幸存区有分为From和To区。当Eden区的空间用完时，程序又需要创建对象，JVM的垃圾回收器将Eden区进行垃圾回收（Minor GC），将Eden区中的不再被其他对象应用的对象进行销毁。然后将Eden区中剩余的对象移到From Survivor区。若From Survivor区也满了，再对该区进行垃圾回收，然后移动到To Survivor区。

##### 2.2老年代（Old Generation）

新生代经过多次GC仍然存活的对象移动到老年区。若老年区也满了，这时候将发生Major GC（也可以叫做Full GC），进行老年区的内存清理。若老年区执行了FullGC之后发现依然无法进行对象的保存，就会抛出OOM（OutOfMemoryError）异常。

##### 2.3元空间（Meta Space）

在JDK1.8之后，元空间替代了永久代，它是对JVM规范中方法区的实现，区别在于元数据区不在虚拟机当中，而是用的本地内存，永久代在虚拟机当中，永久代逻辑结构上也属于堆，但是物理上不属于。

为什么移除了永久代？

参考官方解释：http://openjdk.java.net/jeps/122

大概意思是移除永久代是为融合HotSpot与JRockit而做出的努力，因为JRockit没有永久代，不需要配置永久代。

![image-20191217150436871](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217150436871.png)

#### 3.栈（Stack）

- **虚拟机栈是Java里运行的一个单位，运行的是方法**。

  Java线程执行方法的内存模型，一个线程对应一个栈，每个方法在执行的同时都会创建一个栈帧（用于存储局部变量表，操作数栈，动态链接，方法出口等信息）不存在垃圾回收问题，只要线程一结束该栈就释放，生命周期和线程一致。

  Java的一个线程里就有很多的方法，何况多个线程。

![image-20191216120243219](C:\Users\ginge\Desktop\image-20191216120243219.png)

在一个线程里可能有多个方法，**多个方法对应的是虚拟机栈中的多个栈帧（每一个栈帧对应一个方法）**。栈帧里保存了方法想要的一些东西，比如能够完成这个方法所需的一些操作。Java是自上而下执行的。

堆、方法区可以交给用户自己去进行内存管理，其他三块区域虚拟机不会让用户管。在栈中的方法运行完后就被Java虚拟机回收内存了。也就是说，如果跑完当前栈帧，如果接下来没有要用到的地方，它就可以被回收掉。

- 操作数栈：相当于驿站，可以**临时保管**需要用到的、计算的、返回的参数；跟CPU寄存器交流，把数据丢给CPU去计算，计算完后CPU在把值返回到操作数栈里，再通过操作数栈发到不同地方去如局部变量表。

- 局部变量表：里面都是一些槽，比较抽象，可看作数组，数组里存放的是变量、引用等。

- 返回地址：记录了一个指针，该指针指向了另外一个栈帧（当前这个栈帧即当前这个方法的调用者）。

- 动态链接：在程序运行期间，由符号引用转化为直接引用。

  - 直接引用

    比如代码Object o =  new Object();假设Object是个从没用过的类，跟代码关联的有三个区域：方法区、堆、栈。

    首先创建了Object o的类，这个类是有引用的，这个引用肯定是要放在局部变量表里；这个引用指向了堆里这个唯一实例对象，这个实例对象是个内存地址，这里其实也是一个指针；这个实例对象是根据方法区的一个class文件创建出来的。

    **局部变量表里存放的一个引用指向了堆里的唯一实例对象，这个就叫直接引用**。

  - 符号引用

    ```Java
    javap -v Demo1.class > dynamiclink.txt
    ```

    其结果如下：

    ```
    Classfile /F:/CodeGarden/IdeaProjects/study-jvm/src/main/java/com/jhy/Demo1.class
      Last modified 2019-12-16; size 375 bytes
      MD5 checksum 579e504a1ede63de808039acc381cd01
      Compiled from "Demo1.java"
    public class com.jhy.Demo1
      minor version: 0
      major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #5.#16         // java/lang/Object."<init>":()V
       #2 = Class              #17            // com/jhy/Demo1
       #3 = Methodref          #2.#16         // com/jhy/Demo1."<init>":()V
       #4 = Methodref          #2.#18         // com/jhy/Demo1.math:()I
       #5 = Class              #19            // java/lang/Object
       #6 = Utf8               <init>
       #7 = Utf8               ()V
       #8 = Utf8               Code
       #9 = Utf8               LineNumberTable
      #10 = Utf8               math
      #11 = Utf8               ()I
      #12 = Utf8               main
      #13 = Utf8               ([Ljava/lang/String;)V
      #14 = Utf8               SourceFile
      #15 = Utf8               Demo1.java
      #16 = NameAndType        #6:#7          // "<init>":()V
      #17 = Utf8               com/jhy/Demo1
      #18 = NameAndType        #10:#11        // math:()I
      #19 = Utf8               java/lang/Object
    {
      public com.jhy.Demo1();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 3: 0
    
      public int math();
        descriptor: ()I
        flags: ACC_PUBLIC
        Code:
          stack=2, locals=4, args_size=1
             0: iconst_1
             1: istore_1
             2: iconst_2
             3: istore_2
             4: iload_1
             5: iload_2
             6: iadd
             7: bipush        10
             9: imul
            10: istore_3
            11: iload_3
            12: ireturn
          LineNumberTable:
            line 6: 0
            line 7: 2
            line 8: 4
            line 9: 11
    
      public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
          stack=2, locals=2, args_size=1
             0: new           #2                  // class com/jhy/Demo1
             3: dup
             4: invokespecial #3                  // Method "<init>":()V
             7: astore_1
             8: aload_1
             9: invokevirtual #4                  // Method math:()I
            12: pop
            13: return
          LineNumberTable:
            line 13: 0
            line 14: 8
            line 15: 13
    }
    SourceFile: "Demo1.java"
    ```

  调用方法的指令都是invokevirtual，怎么知道调用的是math方法呢？又找不到#4对应的内容，所以这里需要去打印一些附加的信息。附加信息其实就多了一个东西即constant pool——方法区里的一个常量区（也叫运行时常量池）。

  对照文件查看，#4就是方法的引用，最终指向的是com.jhy.Demo1（全限定名-用来加载类）。#10、#11分别是方法名字和返回类型。任何东西在这个地方分解完了都是字符串也就是一些自变量（这些自变量就是符号引用），通过这些自变量可以找到程序运行的类。在程序运行期间，可能有个类从来没被加载过，想要加载它，要通过在编译期间就确定的你这个类里面需要引用到的一个对象的模板文件存放在哪个地方，这个存在于运行时常量池。在程序运行过程中，通过这一段字符串可以找到堆里的一个实例对象。通过这里就把符号引用转化成了直接引用，再通过直接引用可以找到这个实例对象。

  **动态链接**：在程序运行期间，由符号引用转化为直接引用。

  **静态链接**：在类加载阶段的解析阶段，由符号引用转化为直接引用。

  

#### 4.本地方法栈（Native Method Stack）

本地方法的一个栈结构。和栈作用很相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行native方法服务。登记native方法，在Execution Engine执行时加载本地方法库。

Java里启动线程简析：

![image-20191216163623192](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216163623192.png)

线程启动Thread.start()里调用的start0，表示调用的本地方法，本地方法不存在于Java里面。其实这里就是去调用C++代码，调用的可能是磁盘上的本地动态链接库里的东西。



#### 5.程序计数器（Program Counter Register）

就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址，也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计。

指令集是非常节省的，其中的前面的序号就是程序计数器。**每个线程都有自己的程序计数器，记录程序要运行的下一条将要执行的代码指令的序号。线程要按照正确的顺序去执行，以确保多线程情况下能够正确回到要执行的代码。程序计数器是一块很小的连续的内存空间，这块很小的内存永远不会发生内存溢出、栈异常，因为其内存空间比较小，只放数字（行号指示器），这个数字是块连续的内存空间**。





#### 实战-代码示例分析

编写测试Java类Demo1.java：

```Java
package com.jhy;

public class Demo1 {

    public int math() {
        int a = 1;
        int b = 2;
        int c = (a+b)*10;
        return c;
    }

    public static void main(String[] args) {
        Demo1 demo1 = new Demo1();
        demo1.math();
    }

}
```

Java--->class文件：到文件所在目录，编译java文件,会输出编译结果文件到当前目录

```
javac Demo1.java
```

打开class文件，里面是一堆二进制数据流的十六进制文件，比较复杂。开头的CAFEBABE是个魔术头，防止加载一些非Java内容进来再剔除出去非常麻烦，后面四个小段是JDK版本号。

JDK自带了一个工具**javap，能够对Java代码进行反汇编**。

执行如下命令反汇编class文件并输出到文本文件中：

```java
javap -c Demo1.class > Demo1.txt
```

Demo1.txt内容如下：

所有代码都变成了一行行的指令，**JVM里规定了一整套JVM虚拟机的指令集**，这些指令集负责执行我们的代码。

```java
Compiled from "Demo1.java"
public class com.jhy.Demo1 {
  public com.jhy.Demo1();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public int math();
    Code:
       0: iconst_1
       1: istore_1
       2: iconst_2
       3: istore_2
       4: iload_1
       5: iload_2
       6: iadd
       7: bipush        10
       9: imul
      10: istore_3
      11: iload_3
      12: ireturn

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class com/jhy/Demo1
       3: dup
       4: invokespecial #3                  // Method "<init>":()V
       7: astore_1
       8: aload_1
       9: invokevirtual #4                  // Method math:()I
      12: pop
      13: return
}
```



![image-20191216120243219](C:\Users\ginge\Desktop\image-20191216120243219.png)

对应图和Demo1，当前栈帧就是math方法。



学习上面的指令含义可以去Oracle官网：https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html

搜索iconst到如下结果：

![image-20191216114827827](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216114827827.png)



其中'Push int constant'表示将int类型变量放入栈中，这里是操作数栈。栈是后进先出，进栈出栈都需要经过栈顶的。

搜索istore到如下结果：

![image-20191216145457337](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216145457337.png)

其中'Store int into local variable'表示存储int类型到局部变量表。其实是将iconst中的int**推送**到局部变量表中，

搜索iload到如下结果：

![image-20191216154310642](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216154310642.png)

![image-20191216154324552](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216154324552.png)

iload其实就是从局部变量表里加载int类型的参数。我们说过所有的操作都要经过操作数栈，所以iload从局部变量表里加载出来参数**复制**一份到操作数栈栈顶。

搜索iadd到如下结果：

![image-20191216155031133](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216155031133.png)

iadd就是进行int类型加法。就是从操作数栈中弹出两个值（如果碰到加减乘除运算一定是弹出两个值），CPU计算两个值的加法结果后会把其放到操作数栈栈顶，原始参与加法计算的两个值就被回收了。

bipush就是去常量池里拿一个数字，在Demo1里对应就是10.拿到后会放到操作数栈栈顶。

我们并未给10定义变量，也就是说10并不存在与局部变量表，**局部变量表里找不到对应参数的时候可以去常量池里拿，常量池是存在于方法区的**。

imul乘法计算。



可是为什么Demo1中程序计数器直接从7跳到了9？

因为这里涉及到JVM自身优化，比如说指令重排，其实这里有8的，只不过这里的8被省掉了。8其实就是常量10，常量10也是占了内存地址位的，拿常量的步骤是可以分解的，这里优化成了一个步骤bipush。



后面是istore_3，因为程序又把(a+b)*10的结果赋值给了c，c也是我们定义的一个变量，这个变量也是属于这个方法的，属于这个方法的东西得存到栈帧里的局部变量表里去，加载c回来到操作数栈栈顶，再返回。

这里是通过返回地址进行返回的。返回地址里其实是记录了一个指针，该指针指向了另外一个栈帧（当前这个栈帧即当前这个方法的调用者）。



## GC算法和收集器

### 如何判断对象可以被回收

堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象） 

### 引用计数法

给对象添加一个引用计数器，每当有一个地方引用，计数器就加1。当引用失效，计数器就减1。任何时候计数器为0的对象就是不可能再被使用的。
这个方法实现简单，效率高，但是目前主流的虚拟机中没有选择这个算法来管理内存，主要的原因是它很难解决对象之间相互循环引用的问题。所谓对象之间的相互引用问题，通过下面代码所示：除了对象a和b相互引用着对方之外，这两个对象之间再无任何引用。但是它们因为互相引用对方，导致它们的引用计数器都不为0，于是引用计数器法无法通知GC回收器回收它们。

![image-20191217151944480](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217151944480.png)

### 可达性分析算法

这个算法的基本思想就是通过一系列的称为”GC Roots“的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连的话，则证明此对象是不可用的。GC Roots根节点：类加载器、Thread、虚拟机栈的局部变量表、static成员、常量引用、本地方法栈的变量等等。

![image-20191217152150380](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152150380.png)

### 如何判断一个常量是废弃常量

运行时常量池主要回收的是废弃的常量。那么，我们怎么判断一个常量时废弃常量呢？
假如在常量池中存在字符串"abc"，如果当前没有任何String对象引用该字符串常量的话，就说明常量”abc“就是废弃常量，如果这时发生内存回收的话而且有必要的话，”abc“会被系统清理出常量池。 

### 如何判断一个类是无用的类

需要满足以下三个条件：

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。 

- 加载该类的 ClassLoader 已经被回收。 

- 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

   虚拟机可以对满足上述3个条件的无用类进行回收，这里仅仅是”可以“，而并不是和对象一样不适用了就必然会被回 收。

### 垃圾回收算法

![image-20191217152519552](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152519552.png)



### 标记-清除算法

它是基础的收集算法，这个算法分为两个阶段，“标记”和”清除“。首先标记出所有需要回收的对象，在标记完成后 统一回收所有被标记的对象。它有两个不足的地方：

1. 效率问题，标记和清除两个过程的效率都不高；
2. 空间问题，标记清除后会产生大量不连续的碎片；

![image-20191217152600769](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152600769.png)



### 复制算法

为了解决效率问题，复制算法出现了。它可以把内存分为大小相同的两块，每次只使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块区，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

![image-20191217152632449](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152632449.png)

### 标记-整理算法

根据老年代的特点提出的一种标记算法，标记过程和“标记-清除”算法一样，但是后续步骤不是直接对可回收对象进行回收，而是让所有存活的对象向一段移动，然后直接清理掉边界以外的内存。

![image-20191217152744347](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152744347.png)

### 分代收集算法

现在的商用虚拟机的垃圾收集器基本都采用"分代收集"算法，这种算法就是根据对象存活周期的不同将内存分为几 块。一般将Java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。
在新生代中，每次收集都有大量对象死去，所以可以选择复制算法，只要付出少量对象的复制成本就可以完成每次垃 圾收集。而老年代的对象存活几率时比较高的，而且没有额外的空间对它进行分配担保，就必须选择“标记-清除”或 者“标记-整理”算法进行垃圾收集 。

### 垃圾收集器

Java虚拟机规范对垃圾收集器应该如何实现没有任何规定，因为没有所谓好的垃圾收集器出现，更不会有万金油垃圾收集器，只能是根据具体的应用场景选择合适的垃圾收集器。

![image-20191217152902384](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152902384.png)

### Serial收集器

Serial（串行）收集器收集器是基本、历史悠久的垃圾收集器了。大家看名字就知道这个收集器是一个单线程收 集器了。它的 “单线程” 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ “Stop The World” ），直到它收集结束。 新生代采用复制算法，老年代采用标记-整理算法。

![image-20191217152934806](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217152934806.png)

虚拟机的设计者们当然知道Stop The World带来的不良用户体验，所以在后续的垃圾收集器设计中停顿时间在不断缩短（仍然还有停顿，寻找优秀的垃圾收集器的过程仍然在继续）。

但是Serial收集器有没有优于其他垃圾收集器的地方呢？当然有，它简单而高效（与其他收集器的单线程相比）。 Serial收集器由于没有线程交互的开销，自然可以获得很高的单线程收集效率。

Serial收集器对于运行在Client模式下的虚拟机来说是个不错的选择。

### ParNew收集器

ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集 算法、回收策略等等）和Serial收集器完全一样。 

新生代采用复制算法，老年代采用标记-整理算法。

![image-20191217153040247](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217153040247.png)

它是许多运行在Server模式下的虚拟机的首要选择，除了Serial收集器外，只有它能与CMS收集器（真正意义上的并 发收集器，后面会介绍到）配合工作。 

### Parallel Scavenge收集器（JDK1.8）

Parallel Scavenge 收集器类似于ParNew 收集器。

Parallel Scavenge收集器关注点是吞吐量（高效率的利用CPU）。CMS等垃圾收集器的关注点更多的是用户线程的停 顿时间（提高用户体验）。所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值。 Parallel Scavenge收集器提供了很多参数供用户找到合适的停顿时间或大吞吐量，如果对于收集器运作不太了解的话， 手工优化存在的话可以选择把内存管理优化交给虚拟机去完成也是一个不错的选择。

新生代采用复制算法，老年代采用标记-整理算法。

![image-20191217153123160](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217153123160.png)

### Serial Old收集器

Serial收集器的老年代版本，它同样是一个单线程收集器。它主要有两大用途：一种用途是在JDK1.5以及以前的版本中与Parallel Scavenge收集器搭配使用，另一种用途是作为CMS收集器的后备方案。 

### Parallel Old收集器

Parallel Scavenge收集器的老年代版本。使用多线程和“标记-整理”算法。在注重吞吐量以及CPU资源的场合，都可以优先考虑 Parallel Scavenge收集器和Parallel Old收集器

### CMS收集器

并行和并发概念补充：

- 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。 
- 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个CPU上。

CMS（Concurrent Mark Sweep）收集器是一种以获取短回收停顿时间为目标的收集器。它而非常符合在注重用 户体验的应用上使用。

CMS（Concurrent Mark Sweep）收集器是HotSpot虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。

从名字中的Mark Sweep这两个词可以看出，CMS收集器是一种 “标记-清除”算法实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- 初始标记（CMS initial mark）：暂停所有的其他线程，并记录下直接与root相连的对象，速度很快。
- 并发标记（CMS concurrent mark）：同时开启GC和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- 重新标记（CMS remark）：重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生 变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短。
- 并发清除（CMS concurrent sweep）：开启用户线程，同时GC线程开始对为标记的区域做清扫。

![image-20191217153822886](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217153822886.png)

CMS主要优点：并发收集、低停顿。但是它有下面三个明显的缺点： 

- 对CPU资源敏感； 
- 无法处理浮动垃圾；
- 它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。 

### G1收集器

G1 (Garbage-First)是一款面向服务器的垃圾收集器，主要针对配备多颗处理器及大容量内存的机器.。以极高概率满足GC停顿时间要求的同时，还具备高吞吐量性能特征.

![image-20191217153941544](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217153941544.png)

被视为JDK1.7中HotSpot虚拟机的一个重要进化特征。它具备一下特点：

- 并行与并发：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短StopThe-World停顿时间。部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。
- 分代收集：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。空间整合：与CMS的“标记–清理”算法不同，G1从整体来看是基于“标记整理”算法实现的收集器；从局部上来看是基于“复制”算法实现的。
- 可预测的停顿：这是G1相对于CMS的另一个大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内。

G1收集器的运作大致分为以下几个步骤：

- 初始标记
- 并发标记 
- 最终标记 
- 筛选回收

G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值大的Region(这也就是它的名字Garbage-First的由来)。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了GF收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。 

### 怎么选择垃圾收集器

1. 优先调整堆的大小让服务器自己来选择 
2.  如果内存小于100m，使用串行收集器 
3.  如果是单核，并且没有停顿时间的要求，串行或JVM自己选择 
4.  如果允许停顿时间超过1秒，选择并行或者JVM自己选 
5.  如果响应时间重要，并且不能超过1秒，使用并发收集器

官方推荐G1，性能高。

## JVM性能调优工具

### Jinfo

查看正在运行的Java程序的扩展参数。

#### 查看JVM的参数

![image-20191217155523185](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155523185.png)

#### 查看Java系统属性

等同于System.getProperties()。

![image-20191217155544623](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155544623.png)

### Jstat

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。

命令格式： jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数] 

### Jmap

可以用来查看内存信息 

#### 堆的对象统计

```
jmap -histo 7824 > xxx.txt
```

如图：

![image-20191217155643847](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155643847.png)

- Num：序号 
- Instances：实例数量 
- Bytes：占用空间大小 
- Class Name：类名 

#### 堆信息

![image-20191217155754394](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155754394.png)

#### 堆内存dump

![image-20191217155811477](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155811477.png)

jmap -dump:format=b,ﬁle=temp.hprof

也可以在设置内存溢出的时候自动导出dump文件（项目内存很大的时候，可能会导不出来）

 1.-XX:+HeapDumpOnOutOfMemoryError 

 2.-XX:HeapDumpPath=输出路径

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError XX:HeapDumpPath=d:\oomdump.dump
```

![image-20191217155930331](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155930331.png)

可以使用jvisualvm命令工具导入文件分析

![image-20191217155956416](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217155956416.png)

### Jstack

jstack用于生成java虚拟机当前时刻的线程快照。

![image-20191217160015758](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217160015758.png)

### 调优

JVM调优主要就是调整下面两个指标

停顿时间：垃圾收集器做垃圾回收中断应用执行的时间。-XX:MaxGCPauseMillis 

吞吐量：垃圾收集的时间和总时间的占比：1/(1+n),吞吐量为1-1/(1+n)。-XX:GCTimeRatio=n

#### GC调优步骤

1. 打印GC日志

```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:./gc.log
```

​	Tomcat可以直接加载JAVA_OPTS变量里

   2.分析日志得到关键性指标 

   3.分析GC原因，调优JVM参数 

1.Parallel Scavenge收集器(默认) 

分析parallel-gc.log

第一次调优，设置Metaspace大小：增大元空间大小-XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M

第二次调优，增大年轻代动态扩容增量（默认是20%），可以减少YGC：

-XX:YoungGenerationSizeIncrement=30

比较下几次调优效果：

![image-20191217160240292](C:%5CUsers%5Cginge%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20191217160240292.png)



2.配置CMS收集器

-XX:+UseConcMarkSweepGC

分析gc-cms.log



3.配置G1收集器 

-XX:+UseG1GC 

分析gc-g1.log

young GC:[GC pause (G1 Evacuation Pause)(young) 

initial-mark:[GC pause (Metadata GC Threshold)(young)(initial-mark) (参数：InitiatingHeapOccupancyPercent) 

mixed GC:[GC pause (G1 Evacuation Pause)(Mixed) (参数：G1HeapWastePercent)

full GC:[Full GC (Allocation Failure)(无可用region) 

（G1内部，前面提到的混合GC是非常重要的释放内存机制，它避免了G1出现Region没有可用的情况，否则就会触发FullGC事件。CMS、Parallel、Serial GC都需要通过Full GC去压缩老年代并在这个过程中扫描整个老年代。G1的 Full GC算法和Serial GC收集器完全一致。当一个Full GC发生时，整个Java堆执行一个完整的压缩，这样确保了大的空余内存可用。G1的Full GC是一个单线程，它可能引起一个长时间的停顿时间，G1的设计目标是减少Full GC，满足应用性能目标。）

查看发生MixedGC的阈值：jinfo -ﬂag InitiatingHeapOccupancyPercent 进程ID

调优：

第一次调优，设置Metaspace大小：增大元空间大小-XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M

第二次调优，添加吞吐量和停顿时间参数：-XX:GCTimeRatio=99 -XX:MaxGCPauseMillis=10 

#### GC常用参数

#### 堆栈设置

-Xss:每个线程的栈大小 

-Xms:初始堆大小，默认物理内存的1/64

-Xmx:大堆大小，默认物理内存的1/4 

-Xmn:新生代大小

-XX:NewSize:设置新生代初始大小 

-XX:NewRatio:默认2表示新生代占年老代的1/2，占整个堆内存的1/3。

-XX:SurvivorRatio:默认8表示一个survivor区占用1/8的Eden内存，即1/10的新生代内存。 

-XX:MetaspaceSize:设置元空间大小

-XX:MaxMetaspaceSize:设置元空间大允许大小，默认不受限制，JVM Metaspace会进行动态扩展。 

#### 垃圾回收统计信息

-XX:+PrintGC 

-XX:+PrintGCDetails 

-XX:+PrintGCTimeStamps

-Xloggc:ﬁlename 

#### 收集器设置

-XX:+UseSerialGC:设置串行收集器 

-XX:+UseParallelGC:设置并行收集器

-XX:+UseParallelOldGC:老年代使用并行回收收集器 

-XX:+UseParNewGC:在新生代使用并行收集器 

-XX:+UseParalledlOldGC:设置并行老年代收集器

-XX:+UseConcMarkSweepGC:设置CMS并发收集器 

-XX:+UseG1GC:设置G1收集器

-XX:ParallelGCThreads:设置用于垃圾回收的线程数

#### 并行收集器设置

-XX:ParallelGCThreads:设置并行收集器收集时使用的CPU数。并行收集线程数。

-XX:MaxGCPauseMillis:设置并行收集大暂停时间 

-XX:GCTimeRatio:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n) 

#### CMS收集器设置

-XX:+UseConcMarkSweepGC:设置CMS并发收集器

-XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。 

-XX:ParallelGCThreads:设置并发收集器新生代收集方式为并行收集时，使用的CPU数。并行收集线程数。 

-XX:CMSFullGCsBeforeCompaction:设定进行多少次CMS垃圾回收后，进行一次内存压缩

-XX:+CMSClassUnloadingEnabled:允许对类元数据进行回收 

-XX:UseCMSInitiatingOccupancyOnly:表示只在到达阀值的时候，才进行CMS回收

-XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况

-XX:ParallelCMSThreads:设定CMS的线程数量 

-XX:CMSInitiatingOccupancyFraction:设置CMS收集器在老年代空间被使用多少后触发

-XX:+UseCMSCompactAtFullCollection:设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片的整理 

#### G1收集器设置

-XX:+UseG1GC:使用G1收集器 

-XX:ParallelGCThreads:指定GC工作的线程数量

-XX:G1HeapRegionSize:指定分区大小(1MB~32MB，且必须是2的幂)，默认将整堆划分为2048个分区 

-XX:GCTimeRatio:吞吐量大小，0-100的整数(默认9)，值为n则系统将花费不超过1/(1+n)的时间用于垃圾收集 

-XX:MaxGCPauseMillis:目标暂停时间(默认200ms)

-XX:G1NewSizePercent:新生代内存初始空间(默认整堆5%) 

-XX:G1MaxNewSizePercent:新生代内存大空间

-XX:TargetSurvivorRatio:Survivor填充容量(默认50%) 

-XX:MaxTenuringThreshold:大任期阈值(默认15) 

-XX:InitiatingHeapOccupancyPercen:老年代占用空间超过整堆比IHOP阈值(默认45%),超过则执行混合收集

-XX:G1HeapWastePercent:堆废物百分比(默认5%) 

-XX:G1MixedGCCountTarget:参数混合周期的大总次数(默认8)

****

