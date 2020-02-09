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



