# Java虚拟机内存结构

Java设计之初是用C++设计的，

Java虚拟机（Java Virtual Machine, JVM），一种能够运行Java字节码的虚拟机。作为一种编程语言的虚拟机，实际上不只是专用于Java语言，只要生成的编译文件匹配JVM对加载编译文件格式要求，任何语言都可以由JVM编译运行。

## JVM简介

Sun hotspot，微软vitural machine，阿里巴巴出品的虚拟机，这些虚拟机都是基于Java虚拟机规范的一种实现。不同的虚拟机可以解决各种不同业务场景。

JVM偏向于软件层面的虚拟机，Java虚拟机虽然没有一整套的硬件体系，但是**Java虚拟机有自己独特的指令集、可执行文件结构**，其中可执行的文件结构指的就是class字节码文件。

## JVM基本结构

JVM由三个主要的子系统构成

- 类加载子系统
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

### 运行时数据区

堆、方法区：属于线程共享的。

虚拟机栈、本地方法栈、程序计数器：是本地私有的。

![img](http://ask.qcloudimg.com/http-save/yehe-3596197/on6nhdrdag.png)

#### 堆

- **堆是Java存储的一个单位**；

#### 方法区



#### 虚拟机栈

- **虚拟机栈是Java里运行的一个单位，运行的是方法**。Java的一个线程里就有很多的方法，何况多个线程。

![image-20191216120243219](C:\Users\ginge\Desktop\image-20191216120243219.png)

在一个线程里可能有多个方法，**多个方法对应的是虚拟机栈中的多个栈帧（每一个栈帧对应一个方法）**。Java是自上而下执行的。

堆、方法区可以交给用户自己去进行内存管理，其他三块区域虚拟机不会让用户管。在栈中的方法运行完后就被Java虚拟机回收内存了。也就是说，如果跑完当前栈帧，如果接下来没有要用到的地方，它就可以被回收掉。

- 操作数栈：相当于驿站，可以**临时保管**需要用到的、计算的、返回的参数；跟CPU寄存器交流，把数据丢给CPU去计算，计算完后cpu在把值返回到操作数栈里，再通过操作数栈发到不同地方去。

- 局部变量表：局部变量表里还会放引用。

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

  动态链接！

  

  ​        JIT：运行时编译。

  



#### 程序计数器

指令集是非常节省的，其中的前面的序号就是程序计数器。**每个线程都有自己的程序计数器，记录下一条将要执行的指令的序号。线程要按照正确的顺序去执行，以确保多线程情况下能够正确回到要执行的代码。程序计数器是一块很小的连续的内存空间，这块很小的内存永远不会发生内存溢出、栈异常，因为其内存空间比较小，只放数字（行号指示器），这个数字是块连续的内存空间**。



#### 本地方法栈

本地方法的一个栈结构。

Java里启动线程简析：

![image-20191216163623192](C:\Users\ginge\AppData\Roaming\Typora\typora-user-images\image-20191216163623192.png)

线程启动Thread.start()里调用的start0，表示调用的本地方法，本地方法不存在于Java里面。其实这里就是去调用C++代码，调用的可能是磁盘上的本地动态链接库里的东西。



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









****

