---
title: JAVA知识点汇总
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top: 
tags:
  - java
  - spring
  - mysql
  - http
categories:
  - work
abbrlink: 2de3
date: 2020-10-15 20:50:34
subtitle:
---

## JVM内存模型

### 程序计数器（线程私有）

> 对于一个处理器(如果是多核cpu那就是一核)，在一个确定的时刻都只会执行一条线程中的指令，一条线程中有多个指令，为了线程切换可以恢复到正确执行位置，每个线程都需有独立的一个程序计数器，不同线程之间的程序计数器互不影响，独立存储。

- 当前线程的行号指示器
- 唯一一个不会抛出OutOfMemory的内存区域

### Java虚拟机栈（线程私有）

> 栈描述的是Java方法执行的内存模型。
>
> 每个方法被执行的时候都会创建一个栈帧用于存储局部变量表，操作栈，动态链接，方法出口等信息。每一个方法被调用的过程就对应一个栈帧在虚拟机栈中从入栈到出栈的过程。
>
> 局部变量表所需要的内存空间在编译期完成分配，当进入一个方法时，这个方法在栈中需要分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表大小。

- 每个栈帧存放内容：
    1. 局部变量表（主要）： 方法参数、局部变量、编译器已知的数据类型
    2. 操作数栈
    3. 动态链接
    4. 方法返回地址
- 可能出现两种异常：
    1. 线程请求的栈深度大于虚拟机允许的栈深度，将抛出StackOverflowError。
    2. 虚拟机栈空间可以动态扩展，当动态扩展是无法申请到足够的空间时，抛出OutOfMemory异常。

### 本地方法栈（线程私有）

- 本地方法栈是与虚拟机栈发挥的作用十分相似,区别是虚拟机栈执行的是Java方法(也就是字节码)服务，而本地方法栈则为虚拟机使用到的native方法服务，可能底层调用的c或者c++

### 堆（线程共享）

> 标量替换
>
> 1、标量是指不可分割的量，如java中基本数据类型和reference类型，相对的一个数据可以继续分解，称为聚合量；
>
> 2、如果把一个对象拆散，将其成员变量恢复到基本类型来访问就叫做标量替换；
>
> 3、如果逃逸分析发现一个对象不会被外部访问，并且该对象可以被拆散，那么经过优化之后，并不直接生成该对象，而是在栈上创建若干个成员变量；

- 存放对象实例及数组， 但随着JIT编译器的发展和逃逸分析技术的成熟，这个说法也不是那么绝对(标量替换)
- 大小可固定也可扩展，如果堆中没有内存完成实例分配，而且堆无法扩展将报OOM错误(OutOfMemoryError)

### 方法区（线程共享）

- 用于存储已被虚拟机加载的类信息、常量、静态变量，如static修饰的变量加载类的时候就被加载到方法区中。

## 垃圾回收

### 判断对象是否存活算法

- 引用计数法
    - 原理：通过在对象头中分配一个空间来保存该对象被引用的次数。如果该对象被其它对象引用，则它的引用计数+1，如果删除对该对象的引用，那么它的引用计数就-1，当该对象的引用计数为0时，那么该对象就会被回收。
    - 缺点：循环引用问题---如果有两个对象相互引用，那么这两个对象就不能被回收，因为它们的引用计数始终为1。这也就是我们常说的“内存泄漏”问题。
- 可达性分析法（主流）
    - 原理：从GCroot结点开始向下搜索，路径称为引用链，当对象没有任何一条引用链链接的时候，就认为这个对象是垃圾，并进行回收。
    - 可作为GCroot的对象（GC管理的主要区域是Java堆，一般情况下只针对堆进行垃圾回收。方法区、栈和本地方法区不被GC所管理,因而选择这些区域内的对象作为GC roots,被GC roots引用的对象不被GC回收）：
        1. 虚拟机栈（局部变量表）---局部变量或参数
        2. 方法区中的类静态属性引用的对象
        3. 方法区中的常量引用的对象
        4. 本地方法栈中JNI（native方法）引用的对象

### 如何回收---三大垃圾收集算法

- 1. 标记-清除法

      - 概念：
        1. mutator：应用程序本身，负责NEW(分配内存)、READ(从内存中读取内容)、WRITE(将内容写入内存)
        2. collector：垃圾收集器，回收不再使用的内存，供mutator进行NEW操作
      - 算法原理：标记阶段和清除阶段
        - 标记阶段：collector从mutator根对象开始进行遍历，对从mutator根对象可以访问到的对象都打上一个标识，一般是在对象的header中，将其记录为可达对象。
        - 清除阶段：collector对堆内存(heap memory)从头到尾进行线性的遍历，如果发现某个对象没有标记为可达对象-通过读取对象的header信息，则就将其回收。
        - Tips：Collector在进行标记和清除阶段时会将整个应用程序暂停(mutator)，等待标记清除结束后才会恢复应用程序的运行，这也是Stop-The-World这个单词的来历。
      - 缺点：垃圾收集后有可能会造成大量的内存碎片，假设一个方格代表1个单位的内存，如果有一个对象需要占用3个内存单位的话，那么就会导致Mutator一直处于暂停状态，而Collector一直在尝试进行垃圾收集，直到Out of Memory。

- 2.复制算法

    - 算法原理： 将堆内存对半分为两个半区，只用其中一个半区来进行对象内存的分配，如果在这个半区内存不够给新的对象分配了，那么就开始进行垃圾收集，将这个半区中的所有可达对象都拷贝到另外一个半区中去，然后继续在另外那个半区进行新对象的内存分配。
    - 优点：解决内存碎片问题
    - 缺点：可用堆内存减少了一半
    - 适用场景：回收小的、存活期短的对象，主要针对新生代内存收集方法

- 3.标记-整理算法

    - 算法原理： 标记和整理阶段。

        - 标记阶段与标记清除算法一样
        - 整理阶段：移动所有的可达对象到堆内存的同一个区域中，使他们紧凑的排列在一起，从而将所有非可达对象释放出来的空闲内存都集中在一起，通过这样的方式来达到减少内存碎片的目的。

    - 优点：解决内存碎片问题

    - 缺点：引用额外空间来保存迁移地址，需要遍历多次堆内存

    - 使用场景：主要针对的是老年代内存收集方法

        > 新生代和老年代大小分配： > 响应时间和吞吐量优先的应用，尽可能设置大的新生代和小的老年代

- 分代收集算法

    - 算法原理：根据对象存活的生命周期将内存划分为若干个不同的区域。对不同区域采用不同的回收算法
    - 新生代
        - 新生代包括：
            1. Eden 伊甸园
            2. Survivor 存活区
            3. Tenured Gen 养老区
        - 采取**复制算法**，每次垃圾回收都要回收大部分对象，复制操作较少。一般来说是将新生代划分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden空间和其中的一块Survivor空间，当进行回收时，将Eden和Survivor中还存活的对象复制到另一块Survivor空间中，然后清理掉Eden和刚才使用过的Survivor空间。
    - 老年代
        - 采取**标记整理算法**，每次回收都只回收少量对象

### 如何回收---常见垃圾回收器

- Serial/Serial Old

    - 特点：
        1. 最古老的的垃圾收集器。
        2. 单线程垃圾收集器，在它进行垃圾收集时，必须暂停所有用户线程。
        3. Serial主要针对新生代，采用复制算法。Serial Old针对老年代，采用标记整理算法
    - 优点：简单高效
    - 缺点：停顿

- ParNew

    - 特点：

        ​	Serial的多线程版本

- Parallel Scavenge

    - 特点：
        1. 并行收集器，不需要暂停
        2. 新生代，采用复制算法

- Parallel Old

    - 特点：
        1. Parallel Scavenge老年代版本，并行
        2. 老年代，标记整理算法

- CMS（Concurrent Mark Sweep）

    - 特点：
        1. 并发收集
        2. 采用标记-清除算法
    - 优点：快
    - 缺点：
        1. 占用CPU
        2. 浮动垃圾
        3. 出现ConcurrentMode Failure
        4. 空间碎片

- G1

    - 特点
        1. 当今最前沿的收集器，面向服务端应用
        2. 并行与并发收集器，能建立可预测的停顿时间模型。

按代分类

- 年轻代收集器
    - Serial、ParNew、Parallel Scavenge
- 老年代收集器
    - Serial Old、Parallel Old、CMS收集器
- 特殊收集器
    - G1收集器[新型，不在年轻、老年代范畴内]

## 类加载

### 加载过程：

- 加载

    1. 通过全类名获取定义此类的二进制字节流
    2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
    3. 在内存中生成一个代表该类的 Class 对象,作为方法区这些数据的访问入口
    4. Tips：一个非数组类的加载阶段（获取类的二进制字节流）是可控性最强的阶段，这一步我们可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的 loadClass() 方法）。数组类型不通过类加载器创建，它由 Java 虚拟机直接创建。

- 验证

    - 目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身的安全。 主要包括四种验证：文件格式的验证，元数据的验证，字节码验证，符号引用验证。

        ![image20210918170036312](https://i.loli.net/2021/09/18/F1BQWefNcAYUEDy.png)

- 准备

    - 为类变量（static修饰的字段变量）分配内存并且设置该类变量的初始值，（如static int i = 5 这里只是将 i 赋值为0，在初始化的阶段再把 i 赋值为5)，这里不包含final修饰的static ，因为final在编译的时候就已经分配了。这里不会为实例变量分配初始化，类变量会分配在方法区中，实例变量会随着对象分配到Java堆中。

- 解析

    - 主要的任务是把常量池中的符号引用替换成直接引用，也就是得到类或者字段、方法在内存中的指针或者偏移量。

- 初始化

    - 如果该类具有父类就对父类进行初始化，执行其静态初始化器（静态代码块）和静态初始化成员变量。（前面已经对static 初始化了默认值，这里我们对它进行赋值，成员变量也将被初始化）

        ![image20210918170449021](https://i.loli.net/2021/09/18/RLuVIpE7OWAx4Mz.png)

### 类与类加载器

- 一旦一个类被加载到JVM中，同一个类就不会被再次载入了。
- 在JVM中，一个类用其全类名和其类加载器作为其唯一标识。例如，如果在pg的包中有一个名为Person的类，被类加载器ClassLoader的实例kl负责加载，则该Person类对应的Class对象在JVM中表示为(Person.pg.kl)。
- JVM预定义有三种类加载器
    - 1.根类加载器
        - 加载 Java 的核心类（负责加载$JAVA_HOME中jre/lib/rt.jar里所有的class），由C++实现，不是ClassLoader子类
    - 2.扩展类加载器
        - 它负责加载JRE的扩展目录，lib/ext或者由java.ext.dirs系统属性指定的目录中的JAR包的类。由Java语言实现，父类加载器为null。
    - 3.系统类加载器
        - 它负责在JVM启动时加载CLASSPATH环境变量所指定的JAR包和类路径。通过ClassLoader的静态方法getSystemClassLoader()来获取系统类加载器。用户自定义的类加载器默认以此类加载器作为父加载器。由Java语言实现，父类加载器为ExtClassLoader。

### 类加载机制

- 1.全盘负责

    - 当一个类加载器负责加载某个Class时，该Class所依赖和引用其他Class也将由该类加载器负责载入，除非显式使用另外一个类加载器来载入。

- 2.双亲委派

    - 如果一个类加载器收到了类加载请求，不会自己直接加载，而是把请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载。

        ![image20210918170625728](https://i.loli.net/2021/09/18/tJyd5oZ3aBS1pm9.png)

    - 优势

        - 1.避免类的重复加载。当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。
        - 2.防止核心API库被随意篡改。假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现该类已被加载，则直接返回已加载过的Integer.class。

- 3.缓存机制

    - 该机制保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为什么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。

### forName和loaderClass区别

- Class.forName()得到的class是已经初始化完成的。
- Classloader.loaderClass得到的class是还没有链接（验证，准备，解析三个过程被称为链接）的。

### 对象创建过程

- 在准备实例化一个类的对象前，首先准备实例化该类的父类，依次递归直到递归到Object类。此时，首先实例化Object类，再依次对以下各类进行实例化，直到完成对目标类的实例化。具体而言，在实例化每个类时，都遵循如下顺序：先依次执行实例变量初始化和实例代码块初始化，再执行构造函数初始化。也就是说，编译器会将实例变量初始化和实例代码块初始化相关代码放到类的构造函数中去，并且这些代码会被放在对超类构造函数的调用语句之后，构造函数本身的代码之前。

### 对象的内存布局

- 对象头（markword）

    - 32位系统下，对象头8字节，64位则是16个字节

    - 不同状态下存放数据

        ![image20210918171629651](https://i.loli.net/2021/09/18/5UlmYsTDcFwKuX9.png)

- 实例数据

    - 存放对象程序中各种类型的字段类型，不管是从父类中继承下来的还是在子类中定义的。
    - 分配策略:相同宽度的字段总是放在一起，比如double和long

- 对齐填充

    - 仅起到占位符的作用满足JVM要求。
    - 由于HotSpot规定对象的大小必须是8的整数倍，对象头刚好是整数倍，如果实例数据不是的话，就需要占位符对齐填充。

## synchronized

### 具有可见性和原子性

### JMM关于synchronized的两条规定：

- 线程解锁前，必须把共享变量的最新值刷新到主内存中
- 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时，需要从主内存中重新读取最新的值

### 线程执行互斥代码的过程：

1. 获得互斥锁
2. 清空工作内存
3. 从主内存拷贝变量的最新副本到工作的内存
4. 执行代码
5. 将更改后的共享变量的值刷新到主内存
6. 释放互斥锁

### 锁升级

1. 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁
2. 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1
3. 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。
4. 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁
5. 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
6. 如果自旋成功则依然处于轻量级状态。
7. 如果自旋失败，则升级为重量级锁。

### synchronized 能防止指令重排序吗？

- synchronized不能防止指令重排序，但是能保证有序性，这和volatile实现有序性的方式不同，synchronized是通过互斥锁来保证有序性，即在单线程中无论指令如何重排序，其产生的结果对于其他线程来说是一致的。而volatile是通过内存屏障实现的有序性，即防止指令重排序来保证有序性。

### 死锁

- 两个线程都在等待对方先完成，造成程序的停滞。

- 例如，现在张三想要李四的画，李四想要张三的书，张三对李四说“把你的画给我，我就给你书”，李四也对张三说“把你的书给我，我就给你画”两个人互相等对方先行动，就这么干等没有结果，这实际上就是死锁的概念。示例代码如下：

    ```java
    // 定义张三类
    class Zhangsan {
        public void say() {
            System.out.println("张三对李四说：“你给我画，我就把书给你。”");
        }
    
        public void get() {
            System.out.println("张三得到画了。");
        }
    }
    
    // 定义李四类
    class Lisi {
        public void say() {
            System.out.println("李四对张三说：“你给我书，我就把画给你”");
        }
    
        public void get() {
            System.out.println("李四得到书了。");
        }
    
    }
    
    public class ThreadDeadLock implements Runnable {
    
        private static Zhangsan zs = new Zhangsan();    // 实例化static型对象
    
        private static Lisi ls = new Lisi();    // 实例化static型对象
    
        private boolean flag = false; // 声明标志位，判断那个先说话
    
        public void run() { // 覆写run()方法
            if (flag) {
                synchronized (zs) {  // 同步张三
                    zs.say();
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (ls) {
                        zs.get();
                    }
                }
            } else {
                synchronized (ls) {
                    ls.say();
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (zs) {
                        ls.get();
                    }
                }
            }
        }
    
        public static void main(String args[]) {
            ThreadDeadLock t1 = new ThreadDeadLock();   // 控制张三
            ThreadDeadLock t2 = new ThreadDeadLock();   // 控制李四
            t1.flag = true;
            t2.flag = false;
            Thread thA = new Thread(t1);
            Thread thB = new Thread(t2);
            thA.start();
            thB.start();
        }
    }
    // 程序运行结果： 李四对张三说：“你给我书，我就把画给你” 张三对李四说：“你给我画，我就把书给你。
    ```

- 避免死锁

    - 加锁顺序（线程按照一定的顺序加锁）
    - 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）
    - 死锁检测

## JUC

### volatile

- JMM模型

    ![image20210918171727328](https://i.loli.net/2021/09/18/F23tz5UOqCEwJbA.png)

- 可见性

    - 修改了变量后，新值对其他线程立即可见（基于内存屏障禁止指令重排实现可见性）：
        1. 对volatile变量执行写操作时，会在写操作后加入一条store屏障指令
        2. 对volatile变量执行读操作时，会在读操作前加入一条load屏障指令
    - 线程写volatile变量的过程：
        1. 改变线程工作内存中volatile变量副本的值
        2. 将改变后的副本的值从工作内存刷新到主内存
    - 线程读volatile变量的过程：
        1. 从主内存中读取volatile变量的最新值到线程的工作内存中
        2. 从工作内存中读取volatile变量的副本

- 不具备原子性

    - 如 i++， 不是一个原子性操作，在实际执行时需要三步操作“读-改-写”，在操作未完成前其他线程修改了变量值的话，此操作就无效了
    - 解决方案：
        1. synchronized
        2. ReentrantLock
        3. AtomicInterger

- 适用场合

    - 对变量的写入操作不依赖当前值，如n = n + 1， n ++
    - 该变量没有包含在具有其他变量的不变式中, 如 n < m

- 和synchronizedvolatile的区别

    - volatile不需要加锁，比synchronized更轻量级，不会阻塞线程；
    - 从内存可见性角度，volatile读相当于加锁，volatile写相当于解锁；
    - synchronized既能够保证可见性，又能保证原子性，而volatile只能保证可见性，无法保证原子性。

- double check单例是否需要对实例使用volatile修饰？

    ```java
    public class SingletonClass {
    
        private static SingletonClass instance = null;
        //private static volatile SingletonClass instance = null;
    
        public static SingletonClass getInstance() {
            if (instance == null) {
                synchronized (SingletonClass.class) {
                    if (instance == null) {
                        instance = new SingletonClass();
                    }
                }
            }
            return instance;
        }
    }
    ```

    - 答案是需要，因为**new操作是非原子性的,一般来说包含三个步骤：1.给对象分配内存，2.初始化对象，3.将对象内存地址赋值给引用**，正常来说是1->2->3这么个步骤，但是在指令重排序优化下，由于2、3不存在依赖性，可能会产生1->3>2这样的顺序，即给引用赋值的操作先于初始化操作，那么在多线程环境下，一个线程执行重排序后的指令，刚好执行完给引用赋值这一步，并未进行初始化，另一个线程恰好执行到第一个if语句，此时引用非空，但是实例并没初始化完成，直接返回后，调用实例方法则会发生空指针异常。所以必须依靠volatile来防止重排序，这个时候使用volatile实际上是保证了第一个if读的时候的有序性，对volatile变量的写happen-before读，从而禁止了newSingletonClass()时的重排序。

### Atomic包（CAS）

- 常用类：
    1. AtomicBoolean 、AtomicInteger 、AtomicLong 、 AtomicReference（ 原子引用（声明引用类型的原子类））
    2. AtomicIntegerArray 、AtomicLongArray 3.AtomicStampedReference（原子时间戳引用（根据版本号解决ABA问题））
- 核心方法：
    - boolean compareAndSet(expectedValue, updateValue) 使用CAS思想，依靠Unsafe类的CPU指令原语保证原子性

### Locks包（AQS）

- ReentrantLock（重入锁）
    - 什么是可重入锁：同一线程外层函数获得锁之后，进入内层方法会自动获得锁，即线程可以进入任何一个已经拥有的锁所同步着的代码块
    - ReentrantLock/Synchorized 典型的可重入锁，可重入锁作用：避免死锁
    - lock可重复加，但是加几次就要释放几次，否则会阻塞
- ReadWriteLock（读写锁）

### 并发容器（Collections）

- Queue
    - ConcurrentLinkedQueue
    - BlockingQueue
    - Deque
- CopyOnWriteArraySet
- CopyOnWriteArrayList
- ConcurrentSkipListSet
- ConcurrentMap
    - ConcurrentHashMap
    - ConcurrentNavigableMap
        - ConcurrentSkipListMap

### 执行框架与线程池（Executor）

- Future
    - RunnableFuture
        - RunnableScheduledFuture
        - FutureTask
    - ScheduledFuture
- Callable
- Executor
    - ExecutorService
        - ScheduledExecutorService
            - ScheduledThreadPoolExecutor
        - ThreadPoolExecutor
- CompletionService
    - ExecutorCompletionService
- RejectedExecutionHandler
    - ThreadPoolExecutor.DiscardPolicy
    - ThreadPoolExecutor.DiscardOldestPolicy
    - ThreadPoolExecutor.CallerRunsPolicy
    - ThreadPoolExecutor.AbortPolicy
- TimeUnit

### 并发工具类（Tools）

- CountDownLatch
- CyclicBarrier
- Semaphore
- Executors
- Exchanger

## 集合（TODO 扩容机制）

### Map

- HashMap

    > // 每个数组元素Entery（由key，value，next组成）存储一个链表的头结点Node。 // 当准备添加一个key-value对时，首先通过hash(key)方法计算hash值，然后通过indexFor(hash,length)求该key-value对的存储位置， // 计算方法是先用hash&0x7FFFFFFF(16进制最大正整数按位与)后，再对length取模，这就保证每一个key-value对都能存入HashMap中。 // 没有产生hash冲突前，Node的next是null。当计算出的位置相同时，1.8之前将新的Node插入链表头部，1.8之后，这个链表只让挂7个元素， // 超过七个就会转成一个红黑树进行处理，当红黑树下挂的节点小于等于6的时候，系统会把红黑树转成链表，新的Node插入链表的尾部。

    - 数组+链表(1.8优化成数组+链表+红黑树)的组合实现，数组存储数据，链表解决冲突
    - 线程不安全，继承自AbstractMap类，key可有一个为null，value可有多个null。默认容量为16。
    - jdk1.7和1.8的区别： 1) 1.7使用hash+单链表（头插法），1.8使用hash+链表+红黑树（尾插法，链表长度>7时转成红黑树）。避免出现逆序和链表死循环问题 2) 计算hashcode的方法不同，1.7经过4次位移运算5次异或运算，1.8经过1次位移1次异或 3)扩容时重新计算元素位置的方法不同，1.7重新计算，1.8要么在原位置，要么原位置+扩容大小
    - 扩容为什么是2倍： 只有2的n次幂时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞2的n次幂的情况时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞（最后return的是h&(length-1)，若尾数为0，那么0001、1001、1101等尾数为1的位置就永远不可能被entry占用）

- Hashtable

    > // HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。 // 因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法，可能会进入阻塞或轮询状态。 // 如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

    - Hashtable线程安全，继承自Dictionary类。key和value都不能为null。默认容量为11。

- ConcurrentHashMap

    > // ConcurrentHashMap线程安全，采用分段锁达到高效并发，继承自AbstractMap类。ConcurrentHashMap包含两个静态内部类 HashEntry 和 Segment。 > // HashEntry 用来封装映射表的键 / 值对;Segment 用来充当锁的角色。 > // 一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组，若干个 HashEntry 对象链接起来的链表组成桶。 > // 每个Segment守护着一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。 > // put、get不需要跨段，有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。 > // 这里“按顺序”是很重要的，否则极有可能出现死锁，在ConcurrentHashMap内部，段数组是final的，并且其成员变量实际上也是final的， > // 但是，仅仅是将数组声明为final的并不保证数组成员也是final的，这需要实现上的保证。这可以确保不会出现死锁，因为获得锁的顺序是固定的。

    - ConcurrentHashMap线程安全，采用分段锁达到高效并发，继承自AbstractMap类

- LinkedHashMap

    - LinkedHashMap继承自HashMap，所以它的底层仍然是基于拉链式散列结构，在数组+链表/红黑树的结构上，维护了一条双向链表，保持遍历顺序和插入顺序一致的问题。
    - 线程不安全，增删快。允许有null值null键。

- TreeMap

    - TreeMap实现了SortedMap接口，保证了有序性。线程不安全。不允许有null值null键。
    - 默认的排序是根据key值进行升序排序，也可以重写comparator方法来根据value进行排序具体取决于使用的构造方法。
    - 基于红黑树（Red-Black tree）实现，基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n)。

### List

- ArrayList
    - ArrayList底层结构为数组，线程不安全。查找快，增删慢(除了头尾)。
- Vector
    - Vector底层结构为数组，线程安全(synchronized)。查找慢，增删快。
- LinkedList
    - LinkedList底层结构为双向链表，线程不安全。查找慢，增删快(除了头尾)。

### Set

- HashSet
    - HashSet底层结构为Hash表。无序，无重复值。
- LinkedHashSet
    - LinkedHashSet底层结构为双向链表。有序，无重复值。
    - 继承于HashSet、又基于LinkedHashMap 来实现的
- TreeSet
    - TreeSet底层结构为红黑树。有序，无重复值。效率低。可自定义排序。

### 红黑树

> TreeMap、TreeSet及java8HashMap使用

- 特点：
    1. 自平衡二叉树
    2. 根节点为黑色
    3. 红色节点的两个子节点都是黑色
    4. 任一节点到每个叶子的所有路径都包含相同数目的黑节点
- 插入或删除节点时，规则可能被打破，这时需要动态调整，维持规则（特点3和4）：
    1. 变色
    2. 旋转：左旋、右旋

## spring

### IoC与DI

- IoC：控制反转 反转对象的创建方式，由自己创建反转给程序创建
- DI：依赖注入 不自己定义需要的类，直接向spring容器索取。IoC需要DI支持
- 优点：
    1. 降低组件耦合度
    2. 提供服务，如事务管理，消息处理等。不需要手工控制事务处理复杂的事务传播
    3. 减少代码量

### AOP

### spring工作流

- 解析过程：读xml配置，扫描类文件，从配置或注解中获取bean的定义信息，注册一些扩展功能

- 加载过程：通过解析完的定义信息获取bean实例 获取完整定义 -> 实例化 -> 依赖注入 -> 初始化 -> 类型转换。

    - 1.获取BeanName：对传入的name进行解析，转化为可以从Map中获取到BeanDefinition的bean name

        - 解析完配置后创建的 Map，使用的是 beanName 作为 key，BeanFactory.getBean 中传入的 name，有可能是这几种情况：
            1. bean name，可以直接获取到定义 BeanDefinition。
            2. alias name，别名，需要转化。在解析阶段，alias name 和 bean name 的映射关系被注册到 SimpleAliasRegistry 中。从该注册器中取到 beanName。
            3. factorybean name, 带 & 前缀，通过它获取 BeanDefinition 的时候需要去除 & 前缀。

    - 2.合并 BeanDefinition：对父类的定义进行合并和覆盖，如果父类还有父类，会进行递归合并，以获取完整的 Bean 定义信息。

        - 从配置文件读取到的 BeanDefinition 是 GenericBeanDefinition,在后续实例化 Bean 的时候，使用的是 RootBeanDefinition,如果存在继承关系，GenericBeanDefinition 存储的是 增量信息 而不是 全量信息。在判断 parentName 存在的情况下，说明存在父类定义，启动合并。如果父类还有父类怎么办？递归调用，继续合并。合并完父类定义后，都会调用 RootBeanDefinition.overrideFrom 对父类的定义进行覆盖，获取到当前类能够正确实例化的 全量信息

    - 3.实例化：使用构造或者工厂方法创建 Bean 实例（动态代理+ 反射）。

        - 获取到完整的 RootBeanDefintion 后，就可以拿这份定义信息来实例具体的 Bean。具体实例创建见 AbstractAutowireCapableBeanFactory.createBeanInstance ，返回 Bean 的包装类 BeanWrapper，一共有三种策略：

            1. 使用工厂方法创建，instantiateUsingFactoryMethod 。
            2. 使用有参构造函数创建，autowireConstructor。
            3. 使用无参构造函数创建，instantiateBean。

            三个实例化方式，最后都会走 getInstantiationStrategy().instantiate() 虽然拿到了构造函数，并没有立即实例化。因为用户使用了 replace 和 lookup 的配置方法，用到了动态代理加入对应的逻辑。如果没有的话，直接使用反射来创建实例。 创建实例后，就可以开始注入属性和初始化等操作。

        - 实例化时的**循环依赖问题**，分两种：

            1. 构造器循环依赖。依赖的对象是通过构造器传入的，发生在实例化 Bean 的时候。
            2. 设值循环依赖。依赖的对象是通过 setter 方法传入的，对象已经实例化，发生属性填充和依赖注入的时候。 如果是构造器循环依赖，本质上是无法解决的。比如我们准调用 A 的构造器，发现依赖 B，于是去调用 B 的构造器进行实例化，发现又依赖 C，于是调用 C 的构造器去初始化，结果依赖 A，整个形成一个死结，导致 A 无法创建。

            - 如果是设值循环依赖，Spring 框架只支持单例下的设值循环依赖。Spring 通过对还在创建过程中的单例，缓存并提前暴露该单例，使得其他实例可以引用该依赖。

            - 原型模式的任何循环依赖都不支持 单例模式下，构造函数的循环依赖无法解决，但设值循环依赖是可以解决的（提前暴露创建中的单例）。 为了能够实现单例的提前暴露。Spring 使用了三级缓存，见 DefaultSingletonBeanRegistry 这三个缓存的区别如下：

                1. singletonObjects，单例缓存，存储已经实例化完成的单例。

                2. singletonFactories，生产单例的工厂的缓存，存储工厂。

                3. earlySingletonObjects，提前暴露的单例缓存，这时候的单例刚刚创建完，但还会注入依赖 先尝试从 singletonObjects 和 singletonFactory 读取，没有数据，然后尝试 singletonFactories 读取 singletonFactory，执行 getEarlyBeanReference 获取到引用后，存储到 earlySingletonObjects 中。 这个 earlySingletonObjects 的好处是，如果此时又有其他地方尝试获取未初始化的单例，可以从 earlySingletonObjects 直接取出而不需要再调用 getEarlyBeanReference。 实际上注入 C 的 A 实例，还在填充属性阶段，并没有完全地初始化。等递归回溯回去，A 顺利拿到依赖 B，才会真实地完成 A 的加载。

                    ```java
                    /** Cache of singleton objects: bean name --> bean instance */
                    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
                    
                    /** Cache of singleton factories: bean name --> ObjectFactory */
                    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
                    
                    /** Cache of early singleton objects: bean name --> bean instance */
                    private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
                    ```

    - 4.属性填充：寻找并且注入依赖，依赖的 Bean 还会递归调用 getBean 方法获取。

        - 主要的处理环节有：
            1. 应用 InstantiationAwareBeanPostProcessor 处理器，在属性注入前后进行处理。假设我们使用了 @Autowire 注解，这里会调用到 AutowiredAnnotationBeanPostProcessor 来对依赖的实例进行检索和注入的，它是 InstantiationAwareBeanPostProcessor 的子类。
            2. 根据名称或者类型进行自动注入，存储结果到 PropertyValues 中。
            3. 应用 PropertyValues，填充到 BeanWrapper。这里在检索依赖实例的引用的时候，会递归调用 BeanFactory.getBean 来获得。

    - 5.初始化：调用自定义的初始化方法。

        1. 触发Aware Spring 在初始化阶段，如果判断 Bean 实现了几个Aware接口之一，会往 Bean 中注入它关心的资源。
        2. 触发 BeanPostProcessor（连接IOC和AOP的桥梁） 在 Bean 的初始化前或者初始化后，我们如果需要进行一些增强操作（AOP比如打日志、做校验、属性修改、耗时检测）
        3. 触发自定义 init 自定义初始化有两种方式可以：
            1. 实现 InitializingBean。提供了一个很好的机会，在属性设置完成后再加入自己的初始化逻辑。
            2. 定义 init 方法。自定义的初始化逻辑。

    - 6.获取最终的 Bean：如果是 FactoryBean 需要调用 getObject 方法，如果需要类型转换调用 TypeConverter 进行转化。

### Bean的生命周期

- ![image20210918174436691](https://i.loli.net/2021/09/18/162nF7qvMfN8iKy.png)
- 生命周期过程:
    1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
    2. Bean实例化后将Bean的引入和值注入到Bean的属性中
    3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
    4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
    5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
    6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
    7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
    8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
    9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
    10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

### 事务

- 事务管理方式

    - 编程式事务管理： 代码中调用commit()、rollback()等事务管理方法
    - 声明式事务管理：
        1. 修改spring配置文件，添加事务管理器和事务代理类。
        2. 基于Transactional注解

- 事务特性

    1. 原子性：事务操作要么全部成功，要么全部失败
    2. 一致性：事务执行前后一致，如转账不管转几次，账户总和不变
    3. 隔离性：多个并发事务之间互相隔离
    4. 持久性：事务一旦提交，数据改变是永久性的

- 隔离级别：

    1. 读未提交：事务A读取到事务B未提交的记录，之后B回滚，造成A读取到的数据不是有效的，这种情况称为脏读
    2. 读已提交：事务只能读取到已提交的记录。可避免脏读，但会发生不可重复读。如A两次读取同一条记录的过程中，B修改并提交了该记录，造成A两次结果不一致
    3. 可重复读(mysql默认)：多次从数据库读取某条记录，结果一致。如果多条数据可能会出现幻读。如A读取多条数据，重复两次相同操作，过程中B插进了一条数据，造成A两次读取结果不一致
    4. 串行化：事务执行时会在所有级别上加锁（read、write都加锁），仿佛事务以串行方式进行，性能会大幅下降

- 并发问题

    - 脏读
        - 事务A读取到事务B未提交的记录，之后B回滚，造成A读取到的数据不是有效的
    - 不可重复读
        - 范围在同一条记录。如A两次读取同一条记录的过程中，B修改并提交了该记录，造成A两次结果不一致
    - 幻读
        - 范围在多条记录。如A读取多条数据，重复两次相同操作，过程中B插进了一条数据，造成A两次读取结果不一致

- 传播行为（Propagation）

    ​	// 保证在同一个事务中：

    - 默认Propagation.REQUIRED:支持当前事务，假设当前没有事务。就新建一个事务。

    - SUPPORTS:支持当前事务，假设当前没有事务，就以非事务方式运行。

    - MANDATORY:支持当前事务，假设当前没有事务，就抛出异常。

        // 保证在不同事务中

    - REQUIRES_NEW:新建事务，假设当前存在事务。把当前事务挂起。

    - NOT_SUPPORTED:以非事务方式运行操作。假设当前存在事务，就把当前事务挂起。

    - NEVER:以非事务方式运行，假设当前存在事务，则抛出异常。

    - NESTED:如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。

## mysql

### MyISAM与InnoDB的区别

1. InnoDB支持事务，MyISAM不支持
2. InnoDB支持外键，MyISAM不支持
3. InnoDB是聚集索引，MyISAM是非聚集索引。 聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
4. InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快
5. InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一

### 二叉搜索树

特点:

1. 所有非叶子结点至多有两个儿子
2. 所有结点存储一个关键字
3. 非叶子结点的左指针指向小于其关键字的子树，右指针指向大于其关键字的子树

### B-树

- 多路搜索树

- 特性：

    1. 所有非叶子结点最多只有M个儿子，M>2
    2. 根节点儿子数为【2， M】
    3. 除根结点外的非叶子结点儿子数为【M/2，M】
    4. 每个结点存放【M/2-1，M-1】个关键字（至少2个）
    5. 非叶子结点的关键字个数=指向儿子的指针个数-1；
    6. 非叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；
    7. 非叶子结点的指针：P[1], P[2], …, P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1], K[i])的子树；
    8. 所有叶子结点位于同一层；

    ![image-20210918191232742](https://i.loli.net/2021/09/18/GU86gywBaA7cIiQ.png)

- 总结：

    1. 关键字集合分布在整颗树中；
    2. 任何一个关键字出现且只出现在一个结点中；
    3. 搜索有可能在非叶子结点结束；
    4. 其搜索性能等价于在关键字全集内做一次二分查找；
    5. 自动层次控制；

### B+树

- B+树是B-树的变体，也是一种多路搜索树：

    ![image-20210918191302660](https://i.loli.net/2021/09/18/mV5WLp2BusgXeqr.png)

- 特性：

    1. 非叶子结点的子树指针与关键字个数相同；
    2. 非叶子结点的子树指针P[i]，指向关键字值属于[K[i], K[i+1])的子树（B-树是开区间）；
    3. 为所有叶子结点增加一个链指针；
    4. 所有关键字都在叶子结点出现；

### 三种树对比

- B树：二叉树，每个结点只存储一个关键字，等于则命中，小于走左结点，大于走右结点；
- B-树：多路搜索树，每个结点存储M/2到M个关键字，非叶子结点存储指向关键字范围的子结点；所有关键字在整颗树中出现，且只出现一次，非叶子结点可以命中；
- B+树：在B-树基础上，为叶子结点增加链表指针，所有关键字都在叶子结点中出现，非叶子结点作为叶子结点的索引；B+树总是到叶子结点才命中；

## HTTP

### 三次握手

### 四次挥手

### HTTP1.0和HTTP1.1的区别

- 1.长连接(Persistent Connection)
    - HTTP1.0需要使用keep-alive参数来告知服务器端要建立一个长连接。
    - HTTP1.1支持长连接和请求的流水线处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启长连接keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。
- 2.节约带宽
    - HTTP1.0中存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能。
    - HTTP1.1支持只发送header信息（不带任何body信息），如果服务器认为客户端有权限请求服务器，则返回100，客户端接收到100才开始把请求body发送到服务器；如果返回401，客户端就可以不用发送请求body了节约了带宽。
- 3.HOST域
    - 在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname），HTTP1.0没有host域。随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。
    - HTTP1.1的请求消息和响应消息都支持host域，且请求消息中如果没有host域会报告一个错误（400 Bad Request）。
- 4.缓存处理
    - 在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准
    - HTTP1.1中引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。
- 5.错误通知的管理
    - 在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

### HTTP1.1和HTTP2.0的区别

- 1.多路复用
    - HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。
    - HTTP1.1也可以多建立几个TCP连接，来支持处理更多并发的请求，但是创建TCP连接本身也是有开销的。
- 2.头部数据压缩
    - 在HTTP1.1不支持header数据的压缩，HTTP请求和响应都是由状态行、请求/响应头部、消息主体三部分组成。一般而言，消息主体都会经过gzip压缩，或者本身传输的就是压缩过后的二进制文件，但状态行和头部却没有经过任何压缩，直接以纯文本传输。随着Web功能越来越复杂，每个页面产生的请求数也越来越多，导致消耗在头部的流量越来越多，尤其是每次都要传输UserAgent、Cookie这类不会频繁变动的内容，完全是一种浪费。
    - HTTP2.0使用HPACK算法对header的数据进行压缩，这样数据体积小了，在网络上传输就会更快。
- 3.服务器推送
    - 服务端推送是一种在客户端请求之前发送数据的机制。网页使用了许多资源：HTML、样式表、脚本、图片等等。在HTTP1.1中这些资源每一个都必须明确地请求。这是一个很慢的过程。浏览器从获取HTML开始，然后在它解析和评估页面的时候，增量地获取更多的资源。因为服务器必须等待浏览器做每一个请求，网络经常是空闲的和未充分使用的。
    - 为了改善延迟，HTTP2.0引入了server push，它允许服务端推送资源给浏览器，在浏览器明确地请求之前，免得客户端再次创建连接发送请求到服务器端获取。这样客户端可以直接从本地加载这些资源，不用再通过网络。

### TIME_WAIT和CLOSE_WAIT

- TIME_WAIT：表示主动关闭，通过优化系统内核参数可容易解决。（要么就是对方连接的异常，要么就是自己没有迅速回收资源），解决方法：/etc/sysctl.conf修改，让服务器能够快速回收和重用那些TIME_WAIT的资源。
- CLOSE_WAIT：表示被动关闭，需要从程序本身出发。（在对方关闭连接之后服务器程序自己没有进一步发出ack信号，于是这个资源就一直 被程序占着） ESTABLISHED：表示正在通信

### Https与加密算法

- 密码学在计算机科学中使用非常广泛，HTTPS就是建立在密码学基础之上的一种安全的通信协议
- 对称秘钥:对称密钥加密又叫专用密钥加密，即发送和接收数据的双方必使用相同的密钥对明文进行加密和解密运算。通常有两种模式：流加密和分组加密。
- 非对称秘钥：非对称加密算法需要两个密钥：公开秘钥（publickey）和私有密钥（privatekey）。公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

### session和cookie区别

- 1.存储位置不同
    - cookie的数据信息存放在客户端浏览器上。
    - session的数据信息存放在服务器上。
- 2.存储容量不同
    - 单个cookie保存的数据<=4KB，一个站点最多保存20个Cookie。
    - 对于session来说并没有上限，但出于对服务器端的性能考虑，session内不要存放过多的东西，并且设置session删除机制
- 3.存储方式不同
    - cookie中只能保管ASCII字符串，并需要通过编码方式存储为Unicode字符或者二进制数据。
    - session中能够存储任何类型的数据，包括且不限于string，integer，list，map等。
- 4.隐私策略不同
    - cookie对客户端是可见的，别有用心的人可以分析存放在本地的cookie并进行cookie欺骗，所以它是不安全的。
    - session存储在服务器上，对客户端是透明对，不存在敏感信息泄漏的风险。
- 5.有效期上不同
    - 开发可以通过设置cookie的属性，达到使cookie长期有效的效果。
    - session依赖于名为JSESSIONID的cookie，而cookie JSESSIONID的过期时间默认为-1，只需关闭窗口该session就会失效，因而session不能达到长期有效的效果。
- 6.服务器压力不同
    - cookie保管在客户端，不占用服务器资源。对于并发用户十分多的网站，cookie是很好的选择。
    - session是保管在服务器端的，每个用户都会产生一个session。假如并发访问的用户十分多，会产生十分多的session，耗费大量的内存。
- 7.浏览器支持不同
    - 假如客户端浏览器不支持cookie：
        - cookie是需要客户端浏览器支持的，假如客户端禁用了cookie，或者不支持cookie，则会话跟踪会失效。关于WAP上的应用，常规的cookie就派不上用场了。
        - 运用session需要使用URL地址重写的方式。一切用到session程序的URL都要进行URL地址重写，否则session会话跟踪还会失效。
    - 假如客户端浏览器支持cookie：
        - cookie既能够设为本浏览器窗口以及子窗口内有效，也能够设为一切窗口内有效。
        - session只能在本窗口以及子窗口内有效。
- 8.跨域支持上不同
    - cookie支持跨域名访问。
    - session不支持跨域名访问。

## 分布式事务

## 分布式队列