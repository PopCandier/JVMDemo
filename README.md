### JVM



JVM 在java体系所占的位置。

<https://docs.oracle.com/javase/8/docs/index.html>

![1576119158675](./img/1576119158675.png)

简单的案例是，我们使用`javac`将一个`.java`编译成`.class`文件。

```java
public class Person{
	
	private String name;
	private String age;
	
	public String toString(){
		return name +" "+age;
	}
	
	public Person(String name,String age){
		this.age= age;
		this.name=name;
	}
	
	public static void main(String[] args){
		System.out.println(new Person("Pop","18"));
	}
	
}
```

![1576120202980](./img/1576120202980.png)

用于直接使用文本打开会出现乱码，所以我们使用可以打开二进制文件的工具。`Sublime Text`

```
cafe babe 0000 0034 0036 0700 1d0a 0001
001e 0900 0a00 1f0a 0001 0020 0800 2109
000a 0022 0a00 0100 230a 000f 001e 0900
2400 2507 0026 0800 2708 0028 0a00 0a00
290a 002a 002b 0700 2c01 0004 6e61 6d65
0100 124c 6a61 7661 2f6c 616e 672f 5374
7269 6e67 3b01 0003 6167 6501 0008 746f
5374 7269 6e67 0100 1428 294c 6a61 7661
2f6c 616e 672f 5374 7269 6e67 3b01 0004
436f 6465 0100 0f4c 696e 654e 756d 6265
7254 6162 6c65 0100 063c 696e 6974 3e01
0027 284c 6a61 7661 2f6c 616e 672f 5374
7269 6e67 3b4c 6a61 7661 2f6c 616e 672f
5374 7269 6e67 3b29 5601 0004 6d61 696e
0100 1628 5b4c 6a61 7661 2f6c 616e 672f
5374 7269 6e67 3b29 5601 000a 536f 7572
6365 4669 6c65 0100 0b50 6572 736f 6e2e
....
```

class的文件结构

<https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.1>

```java
/*
 u1表示二个十六进制位
 
 以上面的作为例子，
 cafe babe 下面标注的是 u4，代表u1*4=u4 也就是 八个十六进制，这个是class的规定的
 固定开头，称为魔术位。
*/

ClassFile {
    u4             magic;
    u2             minor_version;//最小版本
    u2             major_version;//最大版本
    u2             constant_pool_count;//常量池数量
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

#### 类加载机制

[Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html)

`.class`文件到JVM的过程

- 装载（loading）
  - 首先找到`.class`文件所在位置，可能是磁盘也可能是网络，`类装载器ClassLoader`
    - ![1581254874930](./img/1581254874930.png)

    - 1）`Bootstrap ClassLoader` 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或 Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。

    -  2）`Extension ClassLoader` 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中 jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。

    -  3）`App ClassLoader` 负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和 jar包。 

    - 4）`Custom ClassLoader` 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据 自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。

    - 利用双亲委派，来保证类加载器加载到jvm只有一个全路径的class对象，先去父亲加载器寻找，如果找不到就往下找，如果都没有，就自己加载。

    - 如何破坏双亲委派。

    - ```java
      // 继承后，重写这个方法，达到破坏 
      // ClassLoader.java
      protected Class<?> loadClass(String name, boolean resolve)
              throws ClassNotFoundException
          {
              synchronized (getClassLoadingLock(name)) {
                  // First, check if the class has already been loaded
                  // 判断这个class是不是早已加载。
                  Class<?> c = findLoadedClass(name);
                  if (c == null) {
                      long t0 = System.nanoTime();
                      try {
                          if (parent != null) {
                              c = parent.loadClass(name, false);//不断往上寻找
                          } else {
                              c = findBootstrapClassOrNull(name);
                          }
                          
      // MyClassLoader.java
      /**
       * @author Pop
       * @date 2020/2/9 21:38
       */
      public class MyClassLoader extends ClassLoader {
          @Override
          public Class<?> loadClass(String name) throws ClassNotFoundException {
              return null;
          }
      }
      
      ```

  - 类文件的信息交给JVM--->不是全部加载，而是将class中的信息，分别存储到不同区域，方法区和堆
  - 类文件所对应的对象class->JVM

- 链接（linking）

  - 验证 （Verify）

    - 保证被加载类的正确性
      - 文件格式验证
      - 元数据验证
      - 字节码验证
      - 符号引用验证

  - 准备 ( Prepare)

    - 为类的静态变量分配内存，并将其初始化为默认值
    - static int a = 10;  --> 先初始化为 0

  - 解析 （ Resolve）

    - 把类中的符号引用转换为直接引用

      - 符号引用，在javac命令编译java文件变成class文件的时候，会产生一个全是十六进制内容的class文件，class文件中含有许多有标志性意义的符号，例如

      - ```java
        ClassFile {
            u4             magic;
            u2             minor_version;//最小版本
            u2             major_version;//最大版本
            u2             constant_pool_count;//常量池数量
            cp_info        constant_pool[constant_pool_count-1];
            u2             access_flags;// 类似这种，符号引用
            u2             this_class;
            u2             super_class;
            u2             interfaces_count;
            u2             interfaces[interfaces_count];
            u2             fields_count;
            field_info     fields[fields_count];
            u2             methods_count;
            method_info    methods[methods_count];
            u2             attributes_count;
            attribute_info attributes[attributes_count];
        }
        ```

      - 他加载到jvm中的时候，变成jvm有意义的内存地址，就是class format，所以我们toString没有重写的话，就会有@6X4F44类似这样的地址。

- 初始化（initializing）

  - 为静态变量初始化真正的值
  - static int a = 0->10



class文件装载到JVM后，JVM会有这样一块区域，用来存储class的信息。

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html

![1581255995083](./img/1581255995083.png)

* **Method Area** :它存储每个**类的结构**，如**运行时常量池**、静态变量，字段和方法数据，以及方法和构造函数的代码，包括在类和实例初始化以及接口初始化中使用的特殊方法，**方法区只有一个，所有线程共享的，非线程安全。生命周期和虚拟机一样**。

* JDK1.7: PermSpace--->永久代
  JDK1.8: MetaSpace---> 元空间

  ```java
  The method area is created on virtual machine start-up.
      // 方法区随着虚拟机启动而创建
      Although the method area is logically part of the heap,
  	// 虽然方法区逻辑上属于堆的一部分
  simple implementations may choose not to either garbage collect or compact it.
      // 垃圾回收不太讨论方法的垃圾回收
      If memory in the method area cannot be made available to satisfy an allocation request, the Java Virtual Machine throws an OutOfMemoryError.
      // 如果方法区没有足够的空间了，那么jvm就会报出 内存溢出异常
  ```

* **Heap** : Java虚拟机有一个在所有Java虚拟机线程之间共享的堆。堆是为所有类**实例和数组分配内存的运行时数据区域**。堆是在虚拟机启动时创建的。对象的堆存储由自动存储管理系统(称为垃圾收集器)回收;对象从不显式释放。Java虚拟机假设没有特定类型的自动存储管理系统，可以根据实现者的系统需求选择存储管理技术。堆的大小可以是固定的，也可以根据计算的需要进行扩展，如果不需要更大的堆，则可以收缩。堆的内存不需要是连续的。

  ```java
  The Java Virtual Machine has a heap that is shared among all Java Virtual Machine threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated.
  // 堆也只有一个，被所有线程共享，非线程安全，生命周期和虚拟机一样，存储对象实例和数组
  If a computation requires more heap than can be made available by the automatic storage management system, the Java Virtual Machine throws an OutOfMemoryError.
      // 如果空间不够，jvm会抛出内存溢出异常
  ```

* **Java Virtual Machine Stacks** ： 可以看出 Method Area 和 Heap 已经将class的信息都存储起来，那么这个区域代表了一个数据结构（栈），先进后出，表示线程调用方法的模型，调用方法时候压栈，低啊用完毕出栈，也就是线程执行方法的表示。上面两个是共享的区域，但是下面就是**每个线程私有**的，每个线程在调用方法时会存在的结构。
  * ![1581258608993](./img/1581258608993.png)

  * ```java
    If the computation in a thread requires a larger Java Virtual Machine stack than is permitted, the Java Virtual Machine throws a StackOverflowError.
        // 栈的深度不够了，就抛出 StackOverflowError
    f Java Virtual Machine stacks can be dynamically expanded, and expansion is attempted but insufficient memory can be made available to effect the expansion, or if insufficient memory can be made available to create the initial Java Virtual Machine stack for a new thread, the Java Virtual Machine throws an OutOfMemoryError.   
    ```

  * `javap`对class文件编译字节码指令，或者 `javap -c Person.class > Person.txt` 导出txt文件。

  * ```java
    public staic int cal(int i,int i2){
        op1 = 3;
        int result = i+i2;
        //Object o = obj;
        return result;
    }
    ```

  * 方法的执行：Frame代表一个栈帧，在一个栈帧中存在，返回值地址

    * Local Variables ： 本地常量表

    * Operand Stacks ： 操作数栈

      * 包含具体数字的计算的压栈和计算完成的出栈

    * Dynamic Linking：动态链接，部分class类型是无法在非运行时期确定的，例如泛型，多态，在正在运行调用的时候才可以确定具体类型。

    * Invocation Completion：调用完成

    * ![1581262493186](./img/1581262493186.png)

    * https://docs.oracle.com/javase/specs/jvms/se8/html/index.html，字节码的意义的解读

    * ```java
      Compiled from "Person.java" 
      class Person { 
      ... public static int calc(int, int); 
          Code:
          	0: iconst_3 //将int类型常量3压入[操作数栈] 
              1: istore_0 //将int类型值存入[局部变量0] 
              2: iload_0 //从[局部变量0]中装载int类型值入栈 
              3: iload_1 //从[局部变量1]中装载int类型值入栈 
              4: iadd //将栈顶元素弹出栈，执行int类型的加法，结果入栈 【For example, the iadd instruction (§iadd) adds two int values together. It requires that the int values to be added be the top two values of the operand stack, pushed there by previous instructions. Both of the int values are popped from the operand stack. They are added, and their sum is pushed back onto the operand stack. Subcomputations may be nested on the operand stack, resulting in values that can be used by the encompassing computation.】 
              5: istore_2 //将栈顶int类型值保存到[局部变量2]中 
              6: iload_2 //从[局部变量2]中装载int类型值入栈 
              7: ireturn //从方法中返回int类型的数据 
          ... 
      }
      ```

    * ![1581262915245](./img/1581262915245.png)

    * **局部变量表**指向**堆**

      ```java
      public int test(int i){
          int result = i+6;
          // obj会存在局部变量表，对象实例存在于 堆
          Object obj = new Object();
          return result;
      }
      ```

      ![1581264973673](./img/1581264973673.png)

    * **方法区**指向**堆**

      ```java
      public static Object obj = new Object();
      // 由于静态变量是存在于 方法区的，对象实例存在堆
      ```

      ![1581264997034](./img/1581264997034.png)

    * **堆**指向**方法区**

      ```java
      // 我们知道 对象实例与数组是存在于堆中的，方法区是存储的class的结构信息
      new Person();//假设我们new了一个Person对象，很明显，他是存在于堆里的，问题是jvm是如何知道他是Person类型的，是由按照Person.class创建出来的？
      // 这就涉及到了Java对象内存布局
      ```

      ![1581265029289](./img/1581265029289.png)从上面我们可以看出，`ClassPointer`就是指向了是谁创建的信息。也就是**堆指向方法区**。![1581265191484](./img/1581265191484.png)
      

* **Native Method Stack**：代表了 java 中的 naive修饰的方法，调用的c的stack

* **The pc Register**

  * 我们都知道一个JVM进程中有多个线程在执行，而线程中的内容是否能够拥有执行权，是根据 

    CPU调度来的。 

    假如线程A正在执行到某个地方，突然失去了CPU的执行权，切换到线程B了，然后当线程A再获 

    得CPU执行权的时候，怎么能继续执行呢？这就是需要在线程中维护一个变量，记录线程执行到 

    的位置。 程序计数器占用的内存空间很小，由于Java虚拟机的多线程是通过线程轮流切换，并分配处理器执行时 

    间的方式来实现的，在任意时刻，一个处理器只会执行一条线程中的指令。因此，为了线程切换后能够 

    **恢复到正确的执行位置**，每条线程需要有一个独立的程序计数器(线程私有)。 

    如果线程正在执行Java方法，则计数器记录的是正在执行的虚拟机字节码指令的地址； 

    如果正在执行的是Native方法，则这个计数器为空。

![1581260268290](./img/1581260268290.png)

#### JVM的内存模型

个人理解的JVM内存模型，由于虚拟机栈和其他区域都是线程私有的，则意味着JVM中如果线程不调用方法，这些结构是不存在的，而**方法区**和**堆**不一样的，他们的生命周期和JVM是相同的，所以当JVM启动的时候，这些区域的内存就已经分配，所以拿这两块来当做JVM的内存模型。MetaSpace和Heap

![1581333894925](./img/1581333894925.png)

* Old区

  * 当一个java对象经过15次(默认)次垃圾回收仍然存活，会被转移到Old区。
  * 一个对象创建的时候过大（100M），会被直接转移到Old区

* Eden+S0+S1

  * GC最频繁的区域
  * S0与S1存在的意义在于对象放在Eden区时候，由于垃圾回收导致存储**不连续**，S0和S1轮流整理这些碎片来保证内存连续，空出更多的空间去存放其它区，S0和S1在同一时间必有一个是空的，这是一种牺牲。
  * 比例上来说，S0**[From]**与S1**[To]**是相同大小，Eden区域分摊剩下的部分，例如Young区是100M，Eden为80M，S0和S1都是10M。Eden:S0:S1=8:1:1
  * 当S0或S1分配空间不够，会向Old区借用一些空间，成为**担保机制**。
  * ![1581334917689](./img/1581334917689.png)

  

![1581335759224](./img/1581335759224.png)

* Young GC [包含了Eden，S区]：Minor GC
* Old GC ：Major GC，MajorGC通常会伴随着MinorGC，差不多类似Full GC
* Young+Old：Full GC 可能还会伴随着 MetaSpace区域GC
* 由于Full GC会带来较长的STW，所以要尽量减少Full GC的频率

#### 垃圾回收

* 一个对象什么时候算作垃圾

  * 引用计数，会有循环引用的问题

  * 可达性分析，GC Root，由它出发，某个对象是否可达。

    * GC Root在Java进程需要很长时间存在，生命周期足够长，不然一下就被回收了作为	Root就没意义了。

    * 可作为CGRoot : 类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法 

      栈的变量等。 

垃圾回收中的吞吐量：用户时间/(用户时间+GC时间)

##### 回收算法

当一个对象已经确定是垃圾的时候，采用何种方式进行回收。

![1581338850072](./img/1581338850072.png)

###### 标记清除（Mark-Sweep）

![1581338948907](./img/1581338948907.png)

先标记，后清除。

```
标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程 序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。 
(1)标记和清除两个过程都比较耗时，效率不高 
(2)会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无 法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
```

###### 复制 （Copying）

为了解决空间的碎片的问题，我们可以参考young区S0和S1的设计，分成两个区域，浪费一半的区域来获得整理的能力。

![1581339262917](./img/1581339262917.png)

左边有垃圾的时候，将活跃对象复制到另一边，复制到另一边的时候便是连续的，左边就可以大胆的全部清空。就会变成上图的样子，左边干净，右边内存连续。

```
空间利用率降低。
```

###### 标记整理（**Mark-Compact**）

标记过程仍然与"标记-清除"算法一样，但是后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

![1581339866365](./img/1581339866365.png)

让所有存活的对象都向一端移动，清理掉边界意外的内存。

![1581340210811](./img/1581340210811.png)

###### 算法的具体落地->垃圾收集器

https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html

不同的垃圾收集器也作用于不同的年龄代。

![1581340441399](./img/1581340441399.png)

撇开垃圾收集器不谈，不同的年龄代适用于什么垃圾回收算法。

* 新生代
  * 适用于复制算法，由于Eden区域需要将存活的对象复制到S1或者S0，来达到内存整理的目的，那么复制的对象不应该过多，不然损耗的性能会比较大。又由于新生代的对象大多数都是朝生夕死，也就意味着需要复制的对象比较少，所以适用于**复制算法**。
* 老年代
  * 老年代不适用于赋值算法，一个是，老年代存在的大小过大的对象，又或者是年龄大于15的对象，所以老年代的对象会存在一种多且大的情况，赋值算法都是比较耗时的，所以适用于**标记整理**或者**标记清除**

垃圾收集器的几个维度

* 单线程/多线程
* 采用什么算法
* 作用于什么代
* 优缺点

例如，Serial垃圾收集器是单线程，采用复制算法，作用于新生代，缺点是单线程可能效率第一点。

垃圾收集器中，需要额外关注的是CMS和G1，其它的区别只是**单线程，多线程，多线程关注吞吐量的**。

###### CMS：Concurrent Mark Sweep 并发垃圾收集器

```java
The Concurrent Mark Sweep (CMS) collector is designed for applications that prefer shorter garbage collection pauses and that can afford to share processor resources with the garbage collector while the application is running. 
//更短的暂停时间 https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector

```

比较关注**停顿时间**。

采用的是"标记-清除算法",整个过程分为4步

```
(1)初始标记 CMS initial mark 标记GC Roots能关联到的对象 Stop The World-- ->速度很快 
(2)并发标记 CMS concurrent mark 进行GC Roots Tracing 
(3)重新标记 CMS remark 修改并发标记因用户程序变动的内容 Stop The World 
(4)并发清除 CMS concurrent sweep 优点：并发收集、低停顿 缺点：产生大量空间碎片
```

![1581342387889](./img/1581342387889.png)

###### G1 

```java
The Garbage-First (G1) garbage collector is a server-style garbage collector, targeted for multiprocessor machines with large memories. It attempts to meet garbage collection (GC) pause time goals with high probability while achieving high throughput. Whole-heap operations, such as global marking, are performed concurrently with the application threads. This prevents interruptions proportional to heap or live-data size.
/*
garbage - first (G1)垃圾收集器是一种服务器风格的垃圾收集器，目标是具有大内存的多处理器机器。它“试图”在实现高吞吐量的同时，以高概率实现垃圾收集(GC)暂停时间目标。整个堆操作(例如全局标记)是与应用程序线程并发执行的。这可以防止与堆或实时数据大小成比例的中断。
https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection
*/
```

> 使用G1收集器时，Java堆的内存布局与就与其他收集器有很大差别，它将整个Java堆划分为多个 大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再 是物理隔离的了，它们都是一部分Region（不需要连续）的集合。

```
初始标记（Initial Marking） 标记一下GC Roots能够关联的对象，并且修改TAMS的值，需要暂 停用户线程 
并发标记（Concurrent Marking） 从GC Roots进行可达性分析，找出存活的对象，与用户线程并发 
执行最终标记（Final Marking） 修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需 暂停用户线程 
筛选回收（Live Data Counting and Evacuation） 对各个Region的回收价值和成本进行排序，根据 用户所期望的GC停顿时间制定回收计划
```

![1581347531047](./img/1581347531047.png)

G1重新对堆进行了布局，为了适用于G1垃圾收集器，按照`Region`，Young和Old只是逻辑上的连续。

![jsgct_dt_004_grbg_frst_hp](./img/jsgct_dt_004_grbg_frst_hp-1581347972137.png)

H表示大对象，S表示Survivor区的内存，他们虽然逻辑上是连续的，也就是S0或S1区确实是存储了数据，但是在上面真实的内容布局中，其实已经被划分成了一块一块的Region独立区域。为何适合G1垃圾收集器。

同时，Region区域可以监控自己区域的使用情况，使用率，还有内部对象大小，然后G1可以根据这些情况来**决定是否回收**，例如太大了，可能回收起来比较麻烦，就下一次再说。

> JDK 7开始使用，JDK 8非常成熟，JDK 9默认的垃圾收集器，适用于新老生代。

判断是否需要使用G1收集器？

```
（1）50%以上的堆被存活对象占用 
（2）对象分配和晋升的速度变化非常大 
（3）垃圾回收时间比较长
```

##### JVM 调优 阶段性小结

对于JVM调优有一个是垃圾收集器的选择

而垃圾收集器有两个维度是比较常用的。停顿时间和吞吐量

* **停顿时间**：停顿时间过长会导致用户线程等待过长，在web程序中相当于用户请求的响应时间过程，我们不希望让用户等待时间太久，时间短可以让用户获得更好的体验，用户的交互也越多，那我们可以选择停段时间小的垃圾收集器
  * CMS、G1[set pause time,not strict 可以设置很短停顿时间，但是不要太严格，不然会频繁触发GC]，他们属于**并发类的收集器**
* **吞吐量**：运行用户代码时间/（运行用户代码时间+垃圾收集时间），则意味着用户代码执行任务占用CPU的时间较长，由于跑任务，运算这些并不怎么需要创建对象和占有内存资源，更多是占用CPU资源，所以更适用于后台跑批任务的程序。
  * Paralled Scanvent(作用于新生代，并行，注重吞吐量的垃圾收集器)+Paralled Old(作用于老年代，并行，注重吞吐量的垃圾收集器) 属于**并行类的收集器**。
* **串行收集**
  * Serial 和 Serial Old 适用于内存比较小，嵌入式的设备

##### 如何查看你的java进程正在什么垃圾收集器

首先启动一个java进程

```
jps -l
```

列出所有的java进程，找到pid

![1581350460275](./img/1581350460275.png)

```
jinfo -flag UseG1GC 16604
```

查看是否使用了G1收集器

![1581350564363](./img/1581350564363.png)

减号，代表没有使用，加号代表正在使用，很明显，这个java进程没有使用G1垃圾收集器。如果你不记得了具体参数名字可以输入

```
jinfo -flags 16604
```

打印出全部JVM参数。

![1581350734220](./img/1581350734220.png)

```java
Attaching to process ID 16604, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.202-b08
Non-default VM flags: 
-XX:-BytecodeVerificationLocal 
-XX:-BytecodeVerificationRemote 
-XX:CICompilerCount=3 
-XX:InitialHeapSize=268435456 
-XX:+ManagementServer 
-XX:MaxHeapSize=4286578688 
-XX:MaxNewSize=1428684800 
-XX:MinHeapDeltaBytes=524288 
-XX:NewSize=89128960 
-XX:OldSize=179306496 
-XX:TieredStopAtLevel=1 
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops 
-XX:+UseFastUnorderedTimeStamps 
-XX:-UseLargePagesIndividualAllocation 
-XX:+UseParallelGC //使用的ParalledGC
Command line:  -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:44529,suspend=y,server=n -XX:TieredStopAtLevel=1 -Xverify:none -Dspring.output.ansi.enabled=always -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=44526 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=localhost -Dspring.liveBeansView.mbeanDomain -Dspring.application.admin.enabled=true -javaagent:C:\Users\99405\.IntelliJIdea2018.3\system\captureAgent\debugger-agent.jar -Dfile.encoding=UTF-8
```

