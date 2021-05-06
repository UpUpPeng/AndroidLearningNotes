# 1. 垃圾回收器 GC

垃圾回收器是 JVM 的三个重要模块（另外是解释器和多线程机制）之一。

为应用程序提供内存的自动分配 (Memory Allocation)、自动回收 (Garbage Collect) 功能，这两个操作都发生在 Java 堆上。

垃圾回收操作需要消耗 CPU、线程、时间等资源，所以当内存消耗完或者是达到某一个指标（Threshold，使用内存占总内存的比列，比如 0.75）时，触发垃圾回收操作。

## 1.1. 确定回收对象的方法

### 1.1.1. 引用计数法

在对象头中分配一个空间来保存该对象被引用的次数，新增一个引用计数+1，释放一个引用计数-1，计数为0时表示无引用可以回收。方法简单，但是不能解决对象相互循环引用的问题。

### 1.1.2. 可达性分析法

通过一些GC Roots对象作为起始节点向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链可以连接时，表示该对象不可用。能解决循环依赖问题。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162155-6f7b706208950968045afa7e65631bce.jpeg)

## 1.2. 可作为GC Root的对象

- 虚拟机栈（栈帧中的局部变量表）中引用的对象，即**正在使用**的引用对象

```java
public static void testGC() {
    Object o = new Object(); // 此时的o就是GC Root
    o = null;                // o置空，则Object对象引用链断掉，将被回收
}
```

- 方法区中类**静态属性**引用的对象

```java
public class Test {
    public static Test t;
}

public static void testGC() {
    Test test = new Test(); // 此时的test就是GC Root
    test.t = new Test();    // t作为类的静态属性，也是GC Root，却依然指向Test对象，因此Test对象不会被回收
    test = null;            // test置空，则Test对象引用链断掉，将被回收
}
```

- 方法区中**常量**引用的对象

```java
public class Test {
    public static final Test t = new Test(); // t即为方法区中的常量引用，也就是GC Root
}

public static void testGC() {
    Test test = new Test();
    test = null; // test置空，则Test对象引用链断掉，将被回收
                 // 但是t所指向的Test对象，也不会因为没有与GC Root建立联系而被回收
}
```

- **本地方法栈**中引用的对象

如果调用的Java方法是一个声明为native的方法时，JVM不会在虚拟机栈中压入新的栈帧，而是简单地动态连接到本地方法栈中对应的Native方法。

## 1.3. 对象的生命周期

在 Java 对象被类加载器加载到虚拟机中后，Java 对象在 Java 虚拟机中有7个阶段。

**创建阶段（Created）**

- 为对象分配存储空间。
- 构造对象。
- 从超类到子类对static成员进行初始化。
- 递归调用超类的构造方法。
- 调用子类的构造方法。

**应用阶段（ln Use）**

当对象被创建，并分配给变量赋值时，状态就切换到了应用阶段。这一阶段的对象至少要具有一个强引用，或者显式地使用软引用、弱引用或者虚引用。

**不可见阶段（Invisible）**

在程序中找不到对象的任何强引用，比如程序的执行已经超出了该对象的作用域。在不可见阶段，对象仍可能被特殊的强引用 GC Roots 持有着，比如对象被本地方法栈中 JNI 引用或被运行中的线程引用等。

**不可达阶段（Unreachable）**

在程序中找不到对象的任何强引用，并且垃圾收集器发现对象不可达。

**收集阶段(Collected)**

垃圾收集器已经发现对象不可达，并且垃圾收集器已经准备好要对该对象的内存空间重新进行分配，这个时候如果该对象重写了 finalize 方法，则会调用该方法。

**终结阶段（Finalized）**

在对象执行完 finalize 方法后仍然处于不可达状态时，或者对象没有重写 finalize 方法，则该对象进入终结阶段，并等待垃圾收集器回收该对象空间。

**对象空间重新分配阶段（Deallocated）**

当垃圾收集器对对象的内存空间进行回收或者再分配时，这个对象就会彻底消失。

## 1.4. 回收时机

### 1.4.1. finalize方法

`finalize()` 是 Object 的 protected 方法，子类可以覆盖该方法以实现资源清理工作，GC 在回收对象之前会调用该方法。

**【用途】**

- 清理本地对象 (通过 JNI 创建的对象)。
- 作为确保某些非内存资源(如 Socket、文件等) 释放的一个补充：在 `finalize()` 方法中显式调用其他资源释放方法。

**【特点】**

- `System.gc()` 与 `System.runFinalization()` 方法会增加 `finalize()` 方法执行的机会，但不绝对。
- Java 语言规范不保证 `finalize()` 方法会被执行和被及时地执行。
- `finalize()` 方法可能会带来性能问题（JVM 通常在单独的低优先级线程中执行 `finalize()` 方法）。
- 对象再生问题：`finalize()` 方法中，可将待回收对象赋值给 GC Roots 可达的对象引用，从而达到对象再生的目的。
- `finalize()` 方法至多由 GC 执行一次（用户可以手动调用，但不影响 GC 对 `finalize()` 方法的行为）。

**【原理】**

1. GC 发生时，如果可回收对象重写了 `finalize()` 方法，则不会立即回收，而是放到 FinalizerQueue 中，反之则直接回收。
2. 专门的 FinalizerThread 会从 FinalizerQueue 取出 finalizer，并执行 `finalize()` 方法。
3. 下一次 GC 发生时，如果该对象与 GCRoots 存在引用链，则不会回收，否则不会再执行 `finalize()` 方法，而是直接回收。

### 1.4.2. 对象复活

```java
public class CanReliveObj {
    public static CanReliveObj obj;

    public static void main(String[] args) throws InterruptedException {
        // 将CanReliveObj对象与GCRoot连接
        obj = new CanReliveObj();
        System.out.println("原始 obj 的哈希值 " + obj.hashCode());

        // 将CanReliveObj对象与GCRoot断开连接
        obj = null;
        System.out.println("第1次GC");
        System.gc();

        Thread.sleep(1000);
        // false。CanReliveObj对象已经复活，不能回收
        System.out.println("obj == null ？" + (obj == null));
        System.out.println("复活 obj 的哈希值 " + obj.hashCode());

        // 将CanReliveObj对象与GCRoot断开连接
        obj = null;
        System.out.println("第2次GC");
        System.gc();

        Thread.sleep(1000);
        // true。finalize方法仅仅会被GC调用一次，不能再复活了
        System.out.println("obj == null ？" + (obj == null));
    }
    
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize方法被调用");
        // 将CanReliveObj对象与GCRoot重新连接，以复活自身对象
        obj = this;
    }

}
```

## 1.5. 垃圾回收方法

JVM并没对如何实现垃圾回收器做出明确规定，因此各个厂商的虚拟机可以有不同的实现方法。

### 1.5.1. 标记清除法

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162654-dbcec754c3fe068a45a7ef472872e1c8.jpeg)

- 最基础的垃圾回收算法。
- 先在内存中把可回收的对象标记出来，再把这些垃圾清理掉变成未使用内存。
- 缺点：**内存碎片**。如上图，会产生一个2M和两个1M的内存，如果申请一个3M的内存空间，则他们都无法使用，因此产生内存碎片。

### 1.5.2. 复制法

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162728-9cfb5827fe6cb78e5b99fedd96673ed2.jpeg)

- 由标记清除算法演化而来，没有内存碎片。
- 将可用内存划分为两块大小相同的区域，每次只用其中的一块；当这块用完了，就将存活对象复制到另一块上，再清除这块的所有空间，保证内存连续可用。
- 缺点：**内存使用率低**。使用率最多达到50%。
- 新生代大多都是生命周期很短的对象，则复制次数较少，因此复制法运用于新生代效率较高。

### 1.5.3. 标记整理法

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162809-769dfbc8cff2e9b1201d5a335ce3a431.jpeg)

- 和标记清除算法类似，没有内存碎片。
- 标记完成之后先把存活对象整理（移动）到同一端，然后再清理掉存活边界以外的内存区域。
- 缺点：**GC效率低**。对内存变动很频繁，由于移动位置后，内存地址会变，所以需要更新引用的指向地址。
- 老年代大多是生命周期很长的对象，因此标记整理法可以降低内存碎片。

### 1.5.4. 分代收集法

融合了以上3中算法的思想，针对不同情况采用不同的算法进行处理。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162839-b535e91433d63a5a575d49b3cb1d80ca.jpeg)

- **Eden区：** 98%的对象生命周期很短，大多数情况下对象都在这里分配，当 Eden 区没有足够空间时，虚拟机会发起 MinorGC ，存活的对象将会移动到 Survivor 的空白子区（不够则直接去 Old 区），然后清空 Eden 区。由于大部分对象都是垃圾，所以 MinorGC 引起的 stop-the-world 可以忽略。

- **Survivor区：** 相当于 Eden 区和 Old 区的缓冲区，分为 From 和 To 两个子区。每次执行 MinorGC，会将 Eden 区和非空子区的存活对象移动到空白子区（空白子区容量不够则直接去 Old 区）。

- - **Survivor区存在的意义：** 减少向 Old 区输送对象，防止 Old 区很快被填满，一般经历过16次 MinorGC 后还存活的对象才会被送到 Old 区。
  - **Survivor区二分的意义：** 为了采用复制法减少内存碎片，From 区和 To 区中的存活对象相互移动，保证任意时刻都有一块子区是空白的。

- **Old区：** 占据2/3的堆内存空间，使用标记整理法执行 MajorGC，清除垃圾内存。Old 区容量越大，MajorGC 影响越大。

**【流程图】**

<img src="https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-162915-9d21e3a701caab933d655f15ebf32eab.jpeg" alt="img" style="zoom:50%;" />

**【内存担保机制】**

有时候无法安置的对象，会直接进入到 Old 区。

- 大对象：需要大量连续内存空间的对象，无论生命周期是长是短，都会直接存入Old区，避免 在Eden 区和 Survivor 区发生大量的内存复制。要重点关注这类大对象。
- 长期存活对象：对象每在 Survivor 区中经历一次 MinorGC 后还存活着，则年龄+1岁，当年龄增加到15（可配置）岁时，便移动到 Old 区。
- 动态对象年龄：如果 Survivor 区中相同年龄的对象大小总和超过 Survivor 区容量的一半，则≥该年龄的对象都会移动到 Old 区，无需等到15岁才移动。

**【相关虚拟机参数】**

| 含义               | 参数                                                         |
| ------------------ | ------------------------------------------------------------ |
| 堆初始大小         | -Xms                                                         |
| 堆最大大小         | -Xmx 或 -XX:MaxHeapSize=size                                 |
| 新生代大小         | -Xmn 或（-XX:NewSize=size 和 -XX:MaxNewSize=size ）          |
| 幸存区比例（动态） | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
| 幸存区比例         | -XX:SurvivorRatio=ratio                                      |
| 晋升阈值           | -XX:MaxTenuringThreshold=threshold                           |
| 晋升详情           | -XX:+PrintTenuringDistribution                               |
| GC详情             | -XX:+PrintGCDetails -verbose:gc                              |
| FullGC前MinorGC    | -XX:+ScavengeBeforeFullGC                                    |

## 1.6. 垃圾收集器

垃圾回收算法是内存回收的抽象策略，垃圾收集器是内存回收的具体实现。

### 1.6.1. Serial 收集器

最基本、历史最悠久的垃圾收集器。（新生代采用复制算法，老生代采用标志整理算法）。大家看名字就知道这个收集器是一个单线程收集器了。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163020-bbb0f0b9327f0b82ef0d2b9ff815f955.jpeg)

**【特点】**

- 单线程，Stop-The-World。
- 新生代-复制法，老生代-标记整理法。
- 简单高效：单 CPU 环境下没有线程调度开销，GC 线程达到最大工作效率。

### 1.6.2. ParNew 收集器

Serial 收集器的多线程版本。除了使用多线程外，其余行为和 Serial 收集器完全一样。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163054-0ef9906652dc14d1de5926282b06cfbb.jpeg)

**【特点】**

- 多线程，Stop The World。
- 新生代-复制法，老生代-标记整理法（不会产生内存碎片）。
- 单 CPU 环境下效率不如 Serial 收集器，多 CPU 环境下效率更高。

### 1.6.3. Parallel Scavenge 收集器

作用于新生代的垃圾收集器。提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，也可以交给虚拟机去完成内存管理优化。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163119-9b8d78862f8966b25595aea72c79721f.jpeg)

**【特点】**

- 只能作用于新生代。
- 多线程，Stop-The-World。
- 新生代-复制法，老生代-标记整理法（不会产生内存碎片）。
- 目标是达到一个可控制的吞吐量。

### 1.6.4. Serial Old 收集器

作用于老年代的垃圾收集器。Serial 收集器的老年代版本。用于在 JDK1.5 及以前的版本中与 Parallel Scavenge 收集器搭配使用，另外也作为 CMS 收集器的后备方案。

**【特点】**

- 只能作用于老年代。
- 单线程，Stop-The-World。
- 新生代-复制法，老生代-标记整理法（不会产生内存碎片）。
- 简单高效：单 CPU 环境下没有线程调度开销，GC 线程达到最大工作效率。

### 1.6.5. Parallel Old 收集器

作用于老年代的垃圾收集器。Parallel Scavenge 收集器的老年代版本。

**【特点】**

- 只能作用于老年代。
- 多线程，Stop-The-World。
- 新生代-复制法，老生代-标记整理法（不会产生内存碎片）。
- 目标是达到一个可控制的吞吐量。

### 1.6.6. CMS 收集器

Concurrent Mark Sweep。作用于老年代的垃圾收集器。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163136-39ac1b2696b6d2682933890c9005bd68.jpeg)

**【特点】**

- 只能作用于老年代。
- 并发收集、低停顿。
- 老生代-标记清除法（不整理，会产生内存碎片）。
- 目标是获取最短回收停顿时间。
- 需要更多的内存。

**【运作过程】**

- 初始标记： Stop-The-World（速度很快）。仅标记 GC Roots 能直接关联到的对象。
- 并发标记：

  - 进行 GC Roots 跟踪的过程。
  - GC 线程和用户线程同时运行，用一个闭包结构去记录可达对象，但不保证包含当前所有的可达对象，所以这个算法里会跟踪记录这些发生引用更新的地方。

- 重新标记： Stop-The-World（比初始标记稍长，远比并发标记短）。修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录（多 GC 线程并行）。
- 并发清除： GC 线程和用户线程同时运行，同时 GC 线程开始对为标记的区域做清扫，回收所有的垃圾对象。

**【缺点】**

- 对 CPU 资源敏感：在并发阶段，它虽然不会导致用户线程停顿，但会因为占用了一部分线程（或者说 CPU 资源）而导致应用程序变慢，总吞吐量会降低。
- 无法处理浮动垃圾：在并发清除时，用户线程新产生的垃圾，称为浮动垃圾。
- 产生大量内存碎片：基于 “标记 + 清除” 算法来回收老年代对象，长时间运行后会产生大量内存碎片，可能会提前触发一次 Full GC。

**【CMS 与 Parallel Old 相比】**

- CMS 减少了执行老年代垃圾收集时应用暂停的时间。
- CMS 增加了新生代垃圾收集时应用暂停的时间、降低了吞吐量而且需要占用更大的堆空间。

### 1.6.7. G1 收集器

Garbage First。 主要针对配备**多处理器**及**大内存**的服务器。以极高概率满足 GC 停顿时间要求的同时，还具备高吞吐量性能特征。JDK1.9 中的默认收集器。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163157-0fd2cb69aa84048c3699d10a5533197c.jpeg)

**【特点】**

- 多线程，Stop-The-World。
- 引入分区的思路，弱化了分代的概念。
- 空间整合：整体上基于标记整理法，局部基于复制法，不会产生内存空间碎片，不会因为分配大对象时无连续空间而提前触发 Full GC。
- 可预测停顿：可以让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒。

**【分代规则】**

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-163212-048439d9ac66d179e77dee5e335b9dce.jpeg)

- 将整个内存区域划分为若干（几千）个大小相等的内存区域小块。
- 每一小块内存区域都有可能是 Eden区、Survivor区、Old区、Humongous区。
- 新对象产生于 Eden 区，回收时将 Eden 区存活对象复制到 Survivor 区，随即该区块成为空白内存区域，其他区块类似。
- 如果一个对象太大，超过了单个区块大小的50%，则会找一个或多个连续的空白区块作为 Humongous 区来存放，不对它进行拷贝操作，并且回收时优先考虑（可以在新生代GC时回收）。

**【运作过程】**

- 初始标记：Stop-The-World（速度很快）。标记出从 GC Root 开始直接可达的对象。
- 并发标记：GC 线程和用户线程同时运行，从 GC Roots 开始对堆中对象进行可达性分析，找出存活对象，耗时较长，但是可以被年轻代垃圾回收打断。
- 最终标记：Stop-The-World。多个 GC 线程并发执行，标记那些在并发标记阶段发生变化的对象。
- 筛选回收：Stop-The-World。多个 GC 线程并发执行，对各个区块的回收价值和成本进行排序，再根据用户所期待的 GC 停顿时间制定回收计划，回收一部分区块。
- 重置：重置线程和用户线程同时运行，整理内存空间，减少内存碎片。

**【G1 的 GC 模式】**

- Young GC：只需要扫描 Eden 区块，将 Eden 区存活对象复制到 Survivor 区或 Old 区（内存担保），然后释放 Eden 区的内存。
- Mixed GC：回收年轻代和部分老年代的区块。如果回收老年代耗时超过了最大暂停时间，则会挑选性价比最高的部分老年代区块进行回收。

------

<br/>

# 2. 引用类型与GC

| **引用类型** | **回收时机** | **用途**   | **生命周期**    |
| :---------: | :--------: | :--------: | :-----------: |
| 强引用       | 手动断开引用 | 重要对象    | 虚拟机后结束     |
| 软引用       | 内存不足时   | 缓存       | 内存不足时结束   |
| 弱引用       | 垃圾回收时   | 缓存       | GC运行完成后结束 |
| 虚引用       | 随时都有可能 | 跟踪回收活动 | 随时可能结束    |

## 2.1. 强引用

大部分引用都是强引用，一般使用 `=` 连接。

```java
Object object = new Object();
String str = "StrongReference";
```

对象具有强引用，内存不足时**不会被回收**，虚拟机抛出 `OutOfMemoryError`。

```java
public class Main {
    public static void main(String[] args) {
        new Main().function();
    }
    public void function() {
        Object[] objArr = new Object[Integer.MAX_VALUE];
    }
}
```

```
Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.renpeng.Main.function(Main.java:9)
	at com.renpeng.Main.main(Main.java:5)
```

## 2.2. 软引用

使用 `SoftReference` 来表示一些有用但不是必需的对象，在**GC后内存还不足时回收**。适合用来实现缓存，如网页缓存、图片缓存。

可以和引用队列（`ReferenceQueue`）结合使用，软引用对象被回收时，虚拟机会把这个软引用加入到引用队列中。

```java
Object obj = new Object();
ReferenceQueue<Object> rq = new ReferenceQueue<>();         // 创建引用队列
SoftReference<Object> sr = new SoftReference<>(obj, rq);    // 建立软引用
System.out.println(sr.get() != null);   // true。没有被回收掉
System.gc();                            // 触发一次GC
System.out.println(sr.get() != null);   // true。内存充足时，GC过后也不会回收掉
```

## 2.3. 弱引用

使用 `WeakReference` 来表示一些可有可无的对象，在**GC时无论内存是否充足都回收**。也可以和引用队列（`ReferenceQueue`）结合使用。

相比软引用，弱引用具有更短的生命周期，不过GC的线程优先级很低，所以并不一定会很快发现弱引用并回收他们。

```java
Object obj = new Object();
ReferenceQueue<Object> rq = new ReferenceQueue<>();     // 创建引用队列
WeakReference<Object> wr = new WeakReference(obj, rq);  // 建立弱引用
// 此时Object对象有两条引用路径，obj强引用和wr弱引用
System.out.println(wr.get() != null);   // true。没有被回收掉
obj = null;                             // 置为null后只剩下wr弱引用一条路径
System.gc();                            // 触发一次GC
System.out.println(wr.get() != null);   // false。GC过后就会被回收掉
```

## 2.4. 虚引用

使用 `PhantomReference` 来表示一些形同虚设的引用对象，在**任何时候都可能回收**。主要用来跟踪对象被垃圾回收的活动。必须和引用队列（`ReferenceQueue`）结合使用。

```java
Object obj = new Object();
ReferenceQueue<Object> rq = new ReferenceQueue<>();             // 创建引用队列
PhantomReference<Object> pr = new PhantomReference<>(obj, rq);  // 建立虚引用
// 此时Object对象有两条引用路径，obj强引用和pr虚引用
System.out.println(pr.get() != null);   // false。get()方法始终返回null
System.out.println(pr.isEnqueued());    // false。虽然get不到，但是它不一定入队了
obj = null;                             // 置为null后只剩下pr虚引用一条路径，GC才能发现
System.gc();                            // 触发一次GC
System.out.println(pr.get() != null);   // false。GC过后，元素只是入队了，依然没有被回收掉
Reference<Object> r = (Reference<Object>) rq.poll();    // 显式调用poll后，元素才被真正回收掉
System.out.println(r != null);          // true。元素被放到了引用队列中
```