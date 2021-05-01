# 1. 基本概念

## 1.1. 进程与线程

### 1.1.1. 二者概念

**【进程】**

- 进程是操作系统结构的基础，是程序在一个数据集合上运行的过程，是**系统进行资源分配和调度的基本单位**。
- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理IO等。
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
- 进程就可以视为程序的一个实例。有的程序可以同时运行多个实例进程（浏览器），有的只能运行一个实例进程（QQ音乐）

**【线程】**

- 线程是操作**系统调度的最小单位**，也叫作轻量级进程。
- 一个进程内可以分为多个线程。
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给CPU执行。
- Java中，线程作为最小调度单位，进程作为资源分配的最小单位。在windows中进程是不活动的，只是作为线程的容器。

### 1.1.2. 二者区别

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集。

- 迸程拥有共享的资源，如内存空间等，供其内部的线程共享。

- 进程间通信较为复杂：

  - 同一台计算机的进程通信称为 IPC (Inter-process communication)。
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如http。

- 线程通信相对简单，因为它们共享进程内的内存，例如多个线程可以访问同一个共享变量。

- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低。

## 1.2. 并行与并发

**【并行 parallel】**

在同一时刻，有多条指令在多个处理器上同时执行。在宏观上和微观上，都是一起执行的。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201201-184230-52617f9d1193e7465fcdbad1a1876225.png)

当系统有一个以上 CPU 时，则线程的操作有可能并行。当一个 CPU 执行一个线程时，另一个 CPU 可以执行另一个线程，两个线程互不抢占 CPU 资源，可以同时进行。

**【并发 concurrency】**

在同一时刻，只有一条指令执行，但多个指令被快速的轮换执行。在宏观上是一起执行的效果，在微观上不是一起执行。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201201-184321-d3f51415b0b3e53bf15b9fc304400284.png)

当系统只有一个 CPU时，则它不可能同时进行一个以上的线程，只能把 CPU 运行时间划分成若干个时间段，再将时间段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状态。

------

<br/>

# 2. 线程

## 2.1. 线程运行的原理

### 2.1.1. 栈与栈帧

JVM 虚拟机在每个线程启动后，就会为其分配一块 Java 虚拟机栈内存。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210106-145039-02eb91cc6fce97f6412f02032982276f.png)

- 每个栈由多个栈帧组成，对应着每次方法调用时所占用的内存。
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法。

### 2.1.2. 线程上下文切换

因为一些原因导致 CPU 不再执行当前的线程，转而执行另一个线程的代码。

- 线程的 CPU 时间片用完。
- 垃圾回收。
- 有更高优先级的线程需要运行。
- 线程自己调用了 `sleep`、`yield`、`wait`、`join`、`park`、`synchronized`、`lock` 等方法。

当上下文切换发生时，需要由操作系统**保存当前线程的状态，并恢复另一个线程的状态**，Java 中程序计数器就是用来记住下一条 JVM 指令的执行地址， 是线程私有的。

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等。
- 上下文切换频繁发生会影响性能。 

## 2.2. 创建线程

### 2.2.1. 直接创建 Thread

创建一个 `Thread` 对象，并重写 `run` 方法。

```java
Thread thread = new Thread("subThread"){
    @Override
    public void run() {
        System.out.println("run subThread!");
    }
};
thread.start();
System.out.println("run main!");
```

> run subThread!
>
> run main!

### 2.2.2. 结合 Runnable

创建一个 `Runnable` 对象，并重写 `run` 方法；再创建一个 `Thread` 对象，并传入 `Runnable` 对象。

⭐ **推荐使用**：能使**任务**和**线程**解耦。

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("run runnable!");
    }
};
Thread thread = new Thread(runnable, "subThread");
thread.start();
System.out.println("run main!");
```

> run main!
>
> run runnable!

### 2.2.3. 结合 FutureTask

创建一个 `FutureTask` 对象，并传入 `Callable` 对象且重写 `call` 方法；再创建一个 `Thread` 对象，并传入 `FutureTask` 对象。

**用途**：`FutureTask` 能够接收 `Callable` 类型的参数，用来处理有返回结果的情况。

```java
FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        System.out.println("run futureTask!");
        return 200;
    }
});
Thread thread = new Thread(futureTask, "subThread");
thread.start();
// 获取任务执行完后的返回值。会阻塞当前线程，直到任务结束返回之后，才继续运行
System.out.println(futureTask.get());
System.out.println("run main!");
```

> run futureTask!
>
> return 200
>
> run main!

## 2.3. 常见方法

### 2.3.1. start() 与 run()

**start()**：

- 启动线程，真正的多线程运行，无需等待 `run()` 方法执行完毕而直接继续执行下面的代码。
- 调用 `start()` 后，线程就处于就绪（可运行）状态，但并没有运行，一旦得到 CPU 时间片，就开始执行 `run() `，`run()` 方法运行结束，此线程随即终止。

```java
Thread thread = new Thread(() -> System.out.println("run subThread!"));
thread.start();
System.out.println("run main!");
```

> run main!
>
> run subThread!

**run()**： 

- 使用当前线程运行 `run()` 方法中的线程体，并不会创建一个线程，因此必须等待 `run()` 方法执行完毕后才能继续执行后面的代码。

```java
Thread thread = new Thread(() -> System.out.println("run subThread!"));
thread.run();
System.out.println("run main!");
```

> run subThread!
>
> run main!

### 2.3.2. sleep() 与 yield()

**sleep():**

- 调用 `sleep()` 会让当前线程从 Running 进入Tined Waiting 状态。
- 其它线程可以使用 `interrupt()` 方法打断正在睡眠的线程，这时 `sleep() `方法会抛出 `InterruptedException`。
- 睡眠结束后的线程未必会立刻得到执行。
- 建议用 `TimeUnit` 的 `sleep()` 代替 `Thread` 的 `sleep()` 来获得更好的可读性。

```java
Thread thread = new Thread(() -> {
    try {
        // 内部进行单位换算，比Thread.sleep(1000)可读性更高
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
System.out.println(thread.getState());  // NEW
thread.start();
// 主线程先运行，则子线程可能还没执行run方法
System.out.println(thread.getState());  // RUNNABLE 或 TIMED_WAITING
// 子线程肯定已经执行到run方法
Thread.sleep(100);
System.out.println(thread.getState());  // TIMED_WAITING
// 子线程在睡眠时被打断，则子线程抛出异常
thread.interrupt();                     // java.lang.InterruptedException: sleep interrupted。
```

**yield()：**

- 调用 `yield()` 会让当前线程从 Running 进入 Runnable 状态，然后调度执行其它同优先级的线程。如果这时没有同优先级的线程，那么不能保证让当前线程暂停的效果。
- 主动让出 CPU 使用权，但如果只有一个线程在运行，则让不出去。具体的实现依赖于操作系统的任务调度器。

**线程优先级：**

- 线程优先级会提示 (hint) 调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它。
- 如果 CPU 比较忙，优先级高的线程会获得更多的时间片，但 CPU 闲时，优先级几乎没作用。

**sleep() 和 yield() 的应用**

防止 CPU 利用率100%。在没有利用 CPU 来计算时（如：线程保活），不要让 `while(true) `空转浪费 CPU。这时可以使用 `yield()` 或 `sleep()` 来让出 CPU 的使用权给其他程序。

```java
new Thread(() -> {
    while (true) {
        try {
            // 防止死循环中，CPU利用率达到100%
            TimeUnit.MILLISECONDS.sleep(50);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).start();
```

### 2.3.3. join()

等待线程运行结束。使**异步并发**变成**同步并发**执行，把多个线程按顺序执行。

```java
static int i = 0;
public static void main(String[] args) throws Exception {
    Thread thread = new Thread(() -> {
        System.out.println("子线程开始");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        i = 100;
        System.out.println("子线程结束");
    });
    thread.start();
    System.out.println(i);
}
```

由于主线程打印 `i` 的时候，子线程还没有对变量 `i` 赋值

> 0
>
> 子线程开始
>
> 子线程结束

```java
thread.start();
thread.join();
System.out.println(i);
```

如果加上 `thread.join();` 则主线程会等待子线程运行结束后，再执行。

> 子线程开始
>
> 子线程结束
>
> 100

### 2.3.4. interrupt()

中断线程。

⭐ **注意：** 仅仅是把线程的运行状态置为中断状态，并不会停止线程。需自己去监视线程的状态（抛出 `InterruptedException` 的方法）为并做处理。

**中断处于阻塞状态的线程：**

sleep、wait、join 导致线程处于阻塞状态。

```java
Thread thread = new Thread(() -> {
    System.out.println("子线程开始");
    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("子线程结束");
});
thread.start();
TimeUnit.SECONDS.sleep(1);
System.out.println("主线程让主线程中断");
thread.interrupt();
// 如果中断是以InterruptedException形式抛出，则isInterrupted为false
// 由于中断已经处理，所以中断标记恢复为false，否则isInterrupted为true
System.out.println("中断标记：" + thread.isInterrupted());
```

> 子线程开始
>
> 主线程让子线程中断
>
> 子线程结束
>
> java.lang.InterruptedException: sleep interrupted
>
> 中断标记：false

**中断处于运行状态的线程：**

```java
Thread thread = new Thread(() -> {
    System.out.println("子线程开始");
    while (true) {
        // isInterrupted()为true，终止死循环，则子线程结束
        if (Thread.currentThread().isInterrupted()) break;
    }
    System.out.println("子线程结束");
});
thread.start();
TimeUnit.SECONDS.sleep(1);
System.out.println("主线程让子线程中断");
thread.interrupt();
TimeUnit.SECONDS.sleep(1);
// 同理，由于子线程内部已经处理了此次中断，所以中断标记恢复为false
System.out.println("中断标记：" + thread.isInterrupted());
```

> 子线程开始
>
> 主线程让子线程中断
>
> 子线程结束
>
> 打断标记：false

**中断处于Park状态的线程：**

```java
Thread thread = new Thread(() -> {
    System.out.println("子线程park");
    LockSupport.park();
    System.out.println("子线程unpark");
    System.out.println("中断标记：" + Thread.currentThread().isInterrupted());
    LockSupport.park(); // 由于中断标记为true，因此再次park无效
    System.out.println("子线程继续运行");
});
thread.start();
TimeUnit.SECONDS.sleep(1);
System.out.println("主线程让子线程中断");
thread.interrupt();
```

> 子线程park
>
> 主线程让子线程中断
>
> 子线程unpark
>
> 中断标记：true
>
> 子线程继续运行

## 2.4. 终止线程-两阶段终止法

A线程优雅的停止B线程，并让B线程完成收尾工作。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210309-153034-5bc4a0703c24bb8e924758ed1b272bb7.png)

```java
Thread thread = new Thread(() -> {
    System.out.println("子线程开始");
    while (true) {
        if (Thread.currentThread().isInterrupted()) {
            // 运行过程中被中断
            System.out.println("执行收尾工作");
            break;
        }
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println("执行循环任务");
        } catch (InterruptedException e) {
            e.printStackTrace();
            // 睡眠过程中被中断
            // 抛出异常后中断标记会置为false，所以需要手动将中断标记设为true，以便下一轮循环时能退出
            Thread.currentThread().interrupt();
        }
    }
    System.out.println("子线程结束");
});
thread.start();
// 三秒钟之后中断线程
TimeUnit.SECONDS.sleep(3);
thread.interrupt();
```

> 子线程开始
>
> 执行循环任务
>
> 执行循环任务
>
> 执行循环任务
>
> java.lang.InterruptedException: sleep interrupted
>
> 执行收尾工作
>
> 子线程结束

## 2.5. 主线程与守护线程

只要有一个线程未结束，进程就不会结束。因此，如果需要主线程运行结束，守护线程（如 GC 线程）也同时结束，则需要将该线程设置为守护线程。

```java
Thread thread = new Thread(() -> {
    while (true) {
        if (Thread.currentThread().isInterrupted()) break;
    }
    // 守护线程在循环中突然结束，后面的代码不会执行。
    System.out.println("守护线程结束");
});
// 设置为守护线程
thread.setDaemon(true);
thread.start();
TimeUnit.SECONDS.sleep(1);
System.out.println("主线程结束");
```

## 2.6. 线程的状态（五种）

从操作系统层面来理解。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210106-175104-2d196f314ae414537f2ab88790626f75.jpg)

1. **新建状态(New):** 线程对象被创建，未与操作系统线程关联。例如 `Thread thread = new Thread()`。

2. **就绪状态(Runnable):** 线程对象被创建后，其它线程调用了该对象的 `start()` 方法来启动该线程，与操作系统线程关联。这种状态随时可能被CPU调度执行。

3. **运行状态(Running):** 线程获取 CPU 权限进行执行。且线程只能从就绪状态进入到运行状态。如果 CPU 时间片用完，又会切换到就绪状态，同时发生线程上线文切换。

4. **阻塞状态(Blocked):** 阻塞状态是线程因为某种原因放弃 CPU 使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种
- 等待阻塞：通过调用线程的 `wait()` 方法，让线程等待某工作的完成。
  
- 同步阻塞：线程在获取 `synchronized` 同步锁失败（锁被其它线程占用）。
  
- 其他阻塞：通过调用线程的 `sleep()` 或 `join()` 或发出了 `I/O` 请求时，线程会进入到阻塞状态。当 `sleep()` 状态超时、`join()` 等待线程终止或者超时、或者 `I/O` 处理完毕时，线程重新转入就绪状态。
  
5. **终止状态(Dead):** 线程正常结束或异常退出了 `run()` 方法，该线程生命周期结束。

## 2.7. 线程的状态（六种）

从 Java API （Thread.State）层面来理解。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210106-191837-df26c1aad63476b567aea72325843fc6.png)

1. **初始(NEW)：** 新创建了一个线程对象，但还没有调用 `start()` 方法。
2. **运行(RUNNABLE)：** Java 线程中将就绪（ready）和运行中（running）两种状态笼统的称为 “运行”。线程对象创建后，其他线程 (比如 main 线程）调用了该对象的 `start()` 方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取 CPU 的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得 CPU 时间片后变为运行中状态（running）。
3. **阻塞(BLOCKED)：** 表示线程阻塞于锁。
4. **等待(WAITING)：** 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. **超时等待(TIMED_WAITING)：** 该状态不同于 WAITING，它可以在指定的时间后自行返回。
6. **终止(TERMINATED)：** 表示该线程已经执行完毕。

```java
// NEW：只创建，但是没有运行
Thread thread1 = new Thread(() -> {
});

// RUNNABLE：一直运行
Thread thread2 = new Thread(() -> {
    while (true) {
    }
});
thread2.start();

// TERMINATED：很快就运行结束了
Thread thread3 = new Thread(() -> {
});
thread3.start();

// TIMED_WAITING：睡眠1小时，计时等待。同时对Main.class加锁
Thread thread4 = new Thread(() -> {
    synchronized (Main.class) {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});
thread4.start();

// WAITING：等待thread2运行完毕，不计时等待
Thread thread5 = new Thread(() -> {
    try {
        thread2.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
thread5.start();

// BLOCKED：thread4已经对Main.class加锁，等待thread4把锁释放
Thread thread6 = new Thread(() -> {
    synchronized (Main.class) {
    }
});
thread6.start();
```

---

<br/>

# 3. 协程

## 3.1. 概念

协程（Coroutine）最直观的解释是**线程中的线程**、**轻量级线程**或者**用户态线程**，一个线程可以拥有多个协程，且**不被操作系统内核管理，完全由程序所控制**（即协程的调度、切换都发生在用户态）。

⭐️ Java 语言并没有对协程的原生支持，但是某些开源框架（[Kilim](https://github.com/kilim/kilim)）模拟出了协程的功能。以后的代码使用 JavaScript 来演示：

**【线程的缺点】**

- 线程中的同步锁是重量级操作。
- 线程在阻塞状态和可运行状态之间切换，需要耗费一定的 CPU 资源。
- 线程的上线文切换，需要耗费一定的 CPU 资源。

**【协程的优点】**

- 不会引起线程上下文切换。
- 协程的开销远远小于线程的开销。

## 3.2. 运行流程

**【普通函数】**

```javascript
function fun() {
    console.log("A"); // A
    console.log("B"); // B
    console.log("C"); // C
}
```

1. 调用 fun。
2. fun 开始执行函数体。
3. fun 执行完成，return。

**【协程函数】**

协程是一个很神奇的函数，它会自己记住之前的执行状态，当再次调用时会从上一次的返回点继续执行。

```javascript
function* fun() {
    yield console.log("A"); 
    // 暂停
    yield console.log("B");
    // 暂停
    yield console.log("C");
}
var gen = fun();
gen.next();	// A
gen.next(); // B
gen.next(); // C
```

**【多协程函数】**

```javascript
function* fun1() {
    yield console.log("A"); 
    // 暂停
    yield console.log("B");
    // 暂停
    yield console.log("C");
}
function* fun2() {
    yield console.log("X"); 
    // 暂停
    yield console.log("Y");
    // 暂停
    yield console.log("Z");
}
var gen1 = fun1();
var gen2 = fun2();
// 两个函数交替执行
gen1.next();	// A
gen2.next();	// X
gen1.next();	// B
gen2.next();	// Y
gen1.next();	// C
gen2.next();	// Z
```

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210118-163435-e947776295a02cd779ffeec22b85c529.png)

协程会在函数被暂停运行时保存函数的运行状态，并可以从保存的状态中恢复并继续运行。

## 3.3. 原理分析

- **理论基础**：每一个方法调用就是方法栈中的一个栈帧，如果使栈顶的栈帧暂停运行并弹出至堆中保存，等需要继续运行时，再从堆压入到栈中继续运行。
- **原理**：在堆中维护一个协程栈区，协程函数不再在方法栈中运行，而是在协程栈中运行，这样就可以随时中断或者恢复协程的执行了。
- **优点**：只要堆区空间足够大，理论上可以使用协程来开启无数并发执行流，同时还没有创建线程的开销，在操作系统看来依然只有一个线程。

