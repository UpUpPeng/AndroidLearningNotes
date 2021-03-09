<!-- TOC -->

- [1. 硬件内存模型](#1-硬件内存模型)
  - [1.1. 数据加载](#11-数据加载)
    - [1.1.1. 处理流程](#111-处理流程)
    - [1.1.2. 缓存行](#112-缓存行)
  - [1.2. 执行流程](#12-执行流程)
- [2. Java 内存模型](#2-java-内存模型)
  - [2.1. 内存划分](#21-内存划分)
    - [2.1.1. 内存模型](#211-内存模型)
    - [2.1.2. 模型类比](#212-模型类比)
  - [2.2. 主内存与工作内存之间的交互操作](#22-主内存与工作内存之间的交互操作)
    - [2.2.1. 个交互协议](#221-个交互协议)
    - [2.2.2. 个基本规则](#222-个基本规则)
  - [2.3. 一致性](#23-一致性)
    - [2.3.1. 原子性](#231-原子性)
    - [2.3.2. 可见性](#232-可见性)
    - [2.3.3. 有序性](#233-有序性)
  - [2.4. 有序性原则](#24-有序性原则)
    - [2.4.1. happens-Before 原则](#241-happens-before-原则)
    - [2.4.2. as-if-serial 语义](#242-as-if-serial-语义)
- [3. volatile 关键字](#3-volatile-关键字)
  - [3.1. 总线嗅探机制](#31-总线嗅探机制)
    - [3.1.1. 缓存一致性协议-MESI](#311-缓存一致性协议-mesi)
    - [3.1.2. 总线嗅探机制](#312-总线嗅探机制)
  - [3.2. 内存屏障](#32-内存屏障)
  - [3.3. 单例模式-双重检查锁](#33-单例模式-双重检查锁)
  - [3.4. volatile 总结](#34-volatile-总结)
- [4. 原子性问题](#4-原子性问题)
  - [4.1. 悲观方案（阻塞同步）](#41-悲观方案阻塞同步)
  - [4.2. 乐观方案（非阻塞同步）](#42-乐观方案非阻塞同步)

<!-- /TOC -->

# 1. 硬件内存模型

按照数据读取顺序和 CPU 的紧密程度，CPU 的缓存可以分为一级缓存（L1）、二级缓存（L2）、三级缓存（L3），每一级缓存存储的数据都是下一级的一部分。    

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-141442-136fd3864df86af57e3870d32b3a1715.png)

## 1.1. 数据加载

### 1.1.1. 处理流程

- 依次从一级缓存、二级缓存、三级缓存中查找，如果没找到再从主内存中查找。
- 把找到的数据依次加载到多级缓存中，下次再使用相关的数据直接从缓存中查找。

### 1.1.2. 缓存行

加载内存中连续的数据，一般来说是加载连续的 64 个字节，因此，如果访问一个 `long` 类型（4个字节）的数组时，当数组中的一个值被加载到缓存中时，另外 7 个元素也会被加载到缓存中，这就是 “缓存行” 的概念。

​    ![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-141442-c6939b9bf9301a340c91115adf125e9c.png)

## 1.2. 执行流程

为了充分利用 CPU 中的运算单元，CPU 可能会对输入的代码进行乱序执行优化，然后在计算之后再将乱序执行的结果进行重组，保证该结果与顺序执行的结果一致，但并不保证程序中各个语句计算的先后顺序与代码的输入顺序一致，因此，如果一个计算任务依赖于另一个计算任务的结果，那么其顺序性并不能靠代码的先后顺序来保证。

------

<br/>

# 2. Java 内存模型

Java 内存模型（Java Memory Model，JMM）是在硬件内存模型基础上更高层的抽象，它屏蔽了各种硬件和操作系统对内存访问的差异性，从而实现让 Java 程序在各种平台下都能达到一致的并发效果。**目的是为了解决多线程环境下共享变量的一致性。**

## 2.1. 内存划分

### 2.1.1. 内存模型

Java 内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存（类比 CPU 的高速缓存）。

​    ![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-141443-1d8b29cdbe1281c487236b09c054df4b.png)

- 工作内存中保存着该线程使用到的变量的主内存副本的拷贝。
- 线程对变量的操作都必须在工作内存中进行，包括读取和赋值等，而不能直接读写主内存中的变量。
- 不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递必须通过主内存来完成。

### 2.1.2. 模型类比

- 主内存：主要对应于硬件内存，堆中对象的实例部分
- 工作内存：主要对应于 CPU 的高速缓存和寄存器部分，虚拟机栈中的部分区域

​    ![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-141444-1bfee926268229760efe7827901a498d.png)

## 2.2. 主内存与工作内存之间的交互操作

### 2.2.1. 个交互协议

- lock，锁定，作用于主内存的变量，它把主内存中的变量标识为一条线程独占状态；
- unlock，解锁，作用于主内存的变量，它把锁定的变量释放出来，释放出来的变量才可以被其它线程锁定；
- read，读取，作用于主内存的变量，它把一个变量从主内存传输到工作内存中，以便后续的 load 操作使用；
- load，载入，作用于工作内存的变量，它把 read 操作从主内存得到的变量放入工作内存的变量副本中；
- use，使用，作用于工作内存的变量，它把工作内存中的一个变量传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；
- assign，赋值，作用于工作内存的变量，它把一个从执行引擎接收到的变量赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时使用这个操作；
- store，存储，作用于工作内存的变量，它把工作内存中一个变量的值传递到主内存中，以便后续的 write 操作使用；
- write，写入，作用于主内存的变量，它把 store 操作从工作内存得到的变量的值放入到主内存的变量中；

操作要按顺序，但不一定要连续

如果要把一个变量从主内存复制到工作内存，那就要按顺序地执行 read 和 load 操作。

如果要把一个变量从工作内存同步回主内存，就要按顺序地执行 store 和 write 操作。

比如，对主内存中的变量 a 和 b 的访问，可以按照以下顺序执行：

read a ->  load a -> read b -> load b，也可以是read a -> read b -> load b -> load a

### 2.2.2. 个基本规则

- 不允许 read 和 load、store 和 write 操作之一单独出现，即不允许出现从主内存读取了而工作内存不接受，或者从工作内存回写了但主内存不接受的情况出现；
- 不允许一个线程丢弃它最近的 assign 操作，即变量在工作内存变化了必须把该变化同步回主内存；
- 不允许一个线程无原因地（即未发生过 assign 操作）把一个变量从工作内存同步回主内存；
- 一个新的变量必须在主内存中诞生，不允许工作内存中直接使用一个未被初始化（load 或 assign）过的变量，换句话说就是对一个变量的 use 和 store 操作之前必须执行过 load 和 assign 操作；
- 一个变量同一时刻只允许一条线程对其进行 lock 操作，但 lock 操作可以被同一个线程执行多次，多次执行 lock 后，只有执行相同次数的 unlock 操作，变量才能被解锁。
- 如果对一个变量执行 lock 操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行 load 或 assign 操作初始化变量的值；
- 如果一个变量没有被 lock 操作锁定，则不允许对其执行 unlock 操作，也不允许 unlock 一个其它线程锁定的变量；
- 对一个变量执行 unlock 操作之前，必须先把此变量同步回主内存中，即执行 store 和 write 操作；

## 2.3. 一致性

一致性主要包含三大特性：原子性、可见性、有序性

### 2.3.1. 原子性

原子性是指一段操作一旦开始就会一直运行到底，中间不会被其它线程打断，这段操作可以是单个或多个操作。

⭐️ **保证指令不会受到上下文切换的影响。**

​    ![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201127-141444-e9946e1674587abf5db1d6a4634e3f27.png)

比如对应一个静态全局变量 int i，两个线程同时对它赋值，线程 A 给他赋值 1，线程 B 给他赋值 - 1。那么 i 的值只能是 1 或者 - 1。线程 A 和线程 B 之间是没有干扰的。这就是原子性的特点，不可被中断。

- 单纯的赋值操作就是原子操作：`a = 1`
- 复合的赋值操作不是原子操作：`a++`

```java
public class ThreadTest {

    private static int count = 0;

    public static void main(String[] args) {
        // 线程1对count自增5000次
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) count++;
        });
		// 线程2对count自减5000次
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) count--;
        });

        thread1.start();
        thread2.start();
    }
}
```

- **理想情况**下，两个线程运行结束后 `count == 0` 。
- **实际情况**下，两个线程运行结束后 `count != 0` 。

`i++` 和 `i--` 在 java 中**不是原子操作**。对于 `i++` 而言（`i` 为静态变量），实际会产生如下的 JVM 字节码指令：

```java
getstatic	i	// 获取静态变量i的值
iconst_1		// 准备常量1
iadd			// 自增
putstatic	i 	// 将修改后的值存入静态变量i
```

而对应 `i--` 也是类似：

```java
getstatic	i 	// 获取静态变量i的值
iconst_1		// 准备常量1
isub			// 自减
putstatic	i 	// 将修改后的值存入静态变量i
```

如果在执行指令的同时，发生了**上下文切换**，则可能一次自增和自减后 `i!=0`。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210309-154324-aa996ceb6a5e581a7687eeec2fe5cf8f.png)

### 2.3.2. 可见性

可见性指当一个线程修改了某一个共享变量的值，其他的线程是否能够立即知道这个修改。

⭐️ **保证指令不会受到 CPU 缓存的影响。**

Java 内存模型是通过在变更修改后同步回主内存，在变量读取前从主内存刷新变量值来实现的，它是依赖主内存的，无论是普通变量还是 volatile 变量都是如此。

```java
public class ThreadTest {

    static boolean run = true;
    public static void main(String[] args) {
        new Thread(() -> {
            // 当run为false时，跳出循环，线程运行结束
            while (run){}
        }).start();

        Thread.sleep(1000);
        // 修改run为false，使线程运行结束
        run = false;
    }
}
```

- **理想情况**下，子线程结束运行 。
- **实际情况**下，子线程仍然运行。 

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210309-154639-6b51729f2dffb5bd785abb5fab86b467.png)

线程2执行 `run = false` 操作。但是这个修改对线程1是不可见的（线程1不知道线程2也修改了 `run` ），那么线程1运算时仍然从缓存中获取 `run`（旧的数据 `run` 为 `true`）。 这时候就出现了数据不一致问题。

**【CPU 主动刷新】**

```java
public class ThreadTest {

    static boolean run = true;
    static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (run){
                // ① println()内部实现有synchronized，所以会每次会刷新
                System.out.println();
                // ② sleep()阻塞了线程，却让CPU有了空闲时间，所以CPU会主动刷新线程的工作内存
                Thread.sleep(100);
                // ③ count++会一直占用CPU，所以CPU不会主动刷新线程的工作内存
                count++;
            }
        }).start();

        Thread.sleep(1000);
        run = false;
    }
}
```

### 2.3.3. 有序性

线程中有序（串行语义），线程间无序（“指令重排序” 现象和 “工作内存和主内存同步延迟” 现象）。因此在并发时，程序的执行可能会出现乱序。给人直观的感觉就是：写在前面的代码，可能会在后面执行。

⭐️ **保证指令不会受到 CPU 指令并行优化的影响。**

**【指令重排】**

为了提高性能，在遵守 `as-if-serial` 语义的情况下，编译器和处理器常常会对指令做重排序。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210309-154736-1639c5b1e53bbc000f595ad11c1809be.png)

分为3种类型：

- 编译器优化重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行重排序。现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

```java
int a = 0;

// 线程 A            可能会做指令重排
a = 1;           // 1 flag = true;
flag = true;     // 2 a = 1;

// 线程 B            如果A发生了指令重排
if (flag) {      // 3 恰好在1和2之间执行，判断flag为true
    int i = a;   // 4 则i为0，不为1，程序发生错误
}   
```

**【为什么要做重排序】**

现代 CPU 支持**多级指令流水线**，例如支持同时执行`取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回`的处理器，就可以称之为**五级指令流水线**。

这时CPU可以在一个时钟周期内，同时运行五条指令的不同阶段（相当于一条执行时间最长的复杂指令），IPC=1，本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了指令地吞吐率。

## 2.4. 有序性原则

### 2.4.1. happens-Before 原则

**先行发生（Happens-Before）**，是指操作 A 先行发生于操作 B，那么操作 A 产生的影响能够被操作 B 感知到，这种影响包括修改了共享内存中变量的值、发送了消息、调用了方法等。

- 程序次序原则：一个线程内保证语义的串行性，准确讲是控制流顺序而不是代码顺序，因为要考虑分支、循环等情况。
- 监视器锁定原则：一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。
- volatile 原则：对一个 volatile 变量的写操作先行发生于后面对该变量的读操作。
- 线程启动原则：对线程的 `start()` 操作先行发生于线程内的任何操作。
- 线程终止原则：线程中的所有操作先行发生于检测到线程终止，可以通过 `Thread.join()`、`Thread.isAlive()` 的返回值检测线程是否已经终止。
- 线程中断原则：线程的中断（ `interrupt()`方法）先于被中断的代码，可以通过 `Thread.interrupted()` 检测是否发生中断。
- 对象终结原则：对象的构造方法执行、结束均优先于 `finalize()`方法。
- 传递性原则：A 先于 B，B 先于 C，那么 A 必然先于 C。

### 2.4.2. as-if-serial 语义

不管怎么重排序，单线程执行结果不能被改变。不会对存在数据依赖关系的操作做重排序，因为这会改变执行结果，但如果操作之间不存在数据依赖关系，就可能被重排序。

```java
double p = 3.14;         // 操作1
double r = 1.0;          // 操作2
double area = p * r * r; // 操作3              
```

1、2步存在指令重排，但是1、2不能与3指令重排，也就是3不可能先于1、2步执行，否则将改变程序的执行结果。

------

<br/>

# 3. volatile 关键字

> https://zhuanlan.zhihu.com/p/138819184

 volatile 关键字能够保证**可见性**和**有序性**，不能保证**原子性**。

主要作用有两点：

- **保证变量的内存可见性**：总线嗅探机制。
- **禁止指令重排序**：内存屏障。

```java
public class VolatileExample {

    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();

        // 主线程执行
        while (true) {
            // 访问子线程变量
            if (myThread.flag) {
                // 永远也不会打印出这句
                // 线程对共享变量的修改没有即时更新到主内存，
                // 或者线程没能够即时将共享变量的最新值同步到工作内存中，
                // 从而使得线程在使用共享变量的值时，该值并不是最新的。
                System.out.println("主线程访问到 flag 变量");
            }
//            解决办法1：使用 synchronizer 进行加锁
//            当一个线程进入 synchronizer 代码块后，线程获取到锁，会清空本地内存，
//            然后从主内存中拷贝共享变量的最新值到本地内存作为副本，执行代码，
//            又将修改后的副本值刷新到主内存中，最后线程释放锁。
//            synchronized (myThread) {
//                if (myThread.isFlag()) {
//                    System.out.println("主线程访问到 flag 变量");
//                }
//            }
        }
    }

    private static class MyThread extends Thread {

        public boolean flag = false;
//        解决办法2：使用 volatile 关键字
//        每个线程要操作变量时会从主内存中将变量拷贝到本地内存作为副本，当线程操作变量副本并写回主内存后，
//        会通过CPU【总线嗅探机制】告知其他线程该变量副本已经失效，需要重新从主内存中读取。
//        private volatile boolean flag = false;

        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 修改变量值
            flag = true;
            System.out.println("修改变量值 flag = true");
        }
    }
}
```

## 3.1. 总线嗅探机制

### 3.1.1. 缓存一致性协议-MESI

早期的 CPU 当中，是通过在总线上直接加锁的形式来解决缓存不一致的问题，但由于在锁住总线期间，其他 CPU 无法访问内存，导致效率低下。

最出名的就是 Intel 的 MESI 协议，MESI 协议保证了每个缓存中使用的共享变量的副本是一致的。

1. **Modify（修改）：** 当缓存行中的数据被修改时，该缓存行置为 M 状态。
2. **Exclusive（独占）：** 当只有一个缓存行使用某个数据时，置为 E 状态。
3. **Shared（共享）：** 当其他 CPU 中也读取某数据到缓存行时，所有持有该数据的缓存行置为 S 状态。
4. **Invalid（无效）：** 当某个缓存行数据修改时，其他持有该数据的缓存行置为 I 状态。

### 3.1.2. 总线嗅探机制

为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，而嗅探是实现缓存一致性的常见机制。

volatile 保证了变量的内存**可见性**：即一个线程修改了 volatile 变量，在写回主内存时，其他线程能立即看到最新值。

- 【原理】每个处理器通过监听在总线上传播的数据来检查自己的缓存值是不是过期了，如果处理器发现自己缓存行对应的内存地址修改，就会将当前处理器的缓存行设置无效状态，当处理器对这个数据进行修改操作的时候，会重新从主内存中把数据读到处理器缓存中。
- 【注意】基于 CPU 缓存一致性协议，JVM 实现了 volatile 的可见性，但由于总线嗅探机制，会不断的监听总线，如果大量使用 volatile 会引起总线风暴。所以，volatile 的使用要适合具体场景。

**【嗅探机制就像是监听器】**

1. CPU1 读取数据 `a=1`，CPU1 的缓存中都有数据 `a` 的副本，该缓存行置为（E）状态。
2. CPU2 也执行读取操作，同样 CPU2 也有数据 `a=1` 的副本，此时总线嗅探到 CPU1 也有该数据，则 CPU1、CPU2 两个缓存行都置为（S）状态。
3. CPU1 修改数据 `a=2`，CPU1 的缓存以及主内存 `a=2`，同时 CPU1 的缓存行置为（S）状态，总线发出通知，CPU2 的缓存行置为（I）状态。
4. CPU2 再次读取 `a`，虽然 CPU2 在缓存中命中数据 `a=1`，但是发现状态为（I），因此直接丢弃该数据，去主内存获取最新数据。

## 3.2. 内存屏障

volatile 保证了本线程内的变量的内存**有序性**（禁止指令重排）：

- **读屏障：** 编译器不会对 volatile 读与后面的任意内存操作重排序。
- **写屏障：** 编译器不会对 volatile 写与前面的任意内存操作重排序。

<html>

<table>
<thead>
  <tr>
    <th colspan="2" rowspan="2">是否重排序</th>
    <th colspan="3">第二次操作</th>
  </tr>
  <tr>
    <td>普通读/写</td>
    <td>volatile 读</td>
    <td>volatile 写</td>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="3">第一次操作</td>
    <td>普通读/写</td>
    <td>⭕️</td>
    <td>⭕️</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>volatile 读</td>
    <td>❌</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
  <tr>
    <td>volatile 写</td>
    <td>⭕️</td>
    <td>❌</td>
    <td>❌</td>
  </tr>
</tbody>
</table>
</html>

## 3.3. 单例模式-双重检查锁

```java
public class Singleton {
    // volatile 保证可见性和禁止指令重排序
    private static volatile Singleton singleton;

    public static Singleton getInstance() {
        // 第一次检查：singleton不为null就直接返回，不需要加锁来消耗多余性能
        if (singleton == null) {
            // 同步代码块：防止第一次初始化的时候造成多例
            synchronized(this.getClass()) {
                // 第二次检查：防止一个线程正在创建实例时，另一个线程阻塞在同步代码块门口，而导致释放锁后又创建一个新对象
                if (singleton == null) {
                    // 对象实例化是非原子性操作：分配内存->初始化实例->返回内存地址给引用
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

**volatile 的重要性**

其中 `singleton = new Singleton();` 是非原子操作，对应的字节码为：

```
new				// ①创建对象，将对象引用入栈
dup				// ②复制一份对象引用（引用地址）
invokeespecial	// ③利用一个对象引用，调用构造方法
putstatic		// ④利用一个对象引用，赋值给 static singleton
```

但是，如果 `static singleton` 没有加 volatile，JVM 可能会做指令重排，将③和④交换执行先后，即 `static singleton` 指向了一个未初始化完毕的对象。此时恰好另一个线程去判断 `if (singleton == null)` 则为 `false`，即返回一个尚未初始化完毕的单例对象 `singleton`。

## 3.4. volatile 总结

- volatile 修饰符适用于：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值；或者作为状态变量，如 `flag = ture`，实现轻量级同步。
- volatile 属性的读写操作都是无锁的，它不能替代 `synchronized`，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的。
- volatile 只能作用于属性，这样编译器就不会对这个属性做指令重排序。
- volatile 提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile 属性不会被线程缓存，始终从主存中读取。
- volatile 提供了 happens-before 保证，对 volatile 变量的写入 happens-before 所有线程后续对他的读操作。
- volatile 可以使纯赋值操作是原子的，如 `boolean flag = true;falg = false` 。
- volatile 可以在单例双重检查中实现可见性和禁止指令重排序，从而保证安全性。

------

<br/>

# 4. 原子性问题

> https://www.cnblogs.com/chengxiao/p/6789109.html

volatile 不能保证对数据操作的原子性，即线程不安全，可以使用锁或者原子类（如 AtomicInteger）。

## 4.1. 悲观方案（阻塞同步）

每次操作数据的时候，都认为其他线程会参与竞争修改，所以使用独占锁机制直接加锁。同一刻只能有一个线程持有锁，其他线程阻塞。线程的挂起恢复会带来很大的性能开销，使用 synchronized 或其他重量级锁来处理显然不够合理。

```java
synchronized(this){
    num++; // 非原子操作：读取->加1->写入
}
```

## 4.2. 乐观方案（非阻塞同步）

每次操作数据的时候，都认为其他线程不会参与竞争修改，所以不加锁。如果操作成功了最好，如果失败也不会阻塞，可以采取一些补偿机制（反复重试）。

```java
// 同样的num++;使用AtomicInteger类的incrementAndGet方式实现

// AtomicInteger.java
public final int incrementAndGet() {
    return U.getAndAddInt(this, VALUE, 1) + 1;
}

// Unsafe.java
@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    // 循环继续尝试更新，直到 weakCompareAndSetInt 返回true
    do {
        // 先获取当前的 value 值
        v = getIntVolatile(o, offset); 
    } while (
        // 进行原子更新操作：
        // 先检查当前 value 是否等于 current。
        // 如果相等，则返回 true，意味着 value 没被其他线程修改过，并更新为目标值。
        // 如果不等，则返回 false。
        !weakCompareAndSetInt(o, offset, v, v + delta)
    );
    return v;
}

@HotSpotIntrinsicCandidate
public final boolean weakCompareAndSetInt(Object o, long offset, int expected, int x) {
    return compareAndSetInt(o, offset, expected, x);
}

// native 方法：使用CAS机器指令，直接保证原子性
@HotSpotIntrinsicCandidate
public final native boolean compareAndSetInt(Object o, long offset, int expected, int x);
```