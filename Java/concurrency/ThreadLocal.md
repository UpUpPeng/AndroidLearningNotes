<!-- TOC -->

- [1. ThreadLocal](#1-threadlocal)
  - [1.1. 作用](#11-作用)
  - [1.2. 使用场景](#12-使用场景)
  - [1.3. 示例代码](#13-示例代码)
- [2. ThreadLocal 原理解析](#2-threadlocal-原理解析)
  - [2.1. 内部设计](#21-内部设计)
    - [2.1.1. 早期方案](#211-早期方案)
    - [2.1.2. JAVA 8 方案](#212-java-8-方案)
  - [2.2. ThreadLocal 核心方法源码](#22-threadlocal-核心方法源码)
    - [2.2.1. set 方法](#221-set-方法)
    - [2.2.2. get 方法](#222-get-方法)
    - [2.2.3. remove方法](#223-remove方法)
- [3. ThreadLocalMap 源码分析](#3-threadlocalmap-源码分析)
  - [3.1. ThreadLocalMap 类的基本结构](#31-threadlocalmap-类的基本结构)
    - [3.1.1. 成员变量](#311-成员变量)
    - [3.1.2. 存储元素](#312-存储元素)
  - [3.2. ThreadLocal 内存泄漏](#32-threadlocal-内存泄漏)
    - [3.2.1. 当key是强引用](#321-当key是强引用)
    - [3.2.2. 当key是弱引用](#322-当key是弱引用)
    - [3.2.3. 导致内存泄漏的原因](#323-导致内存泄漏的原因)
  - [3.3. 解决Hash冲突](#33-解决hash冲突)
    - [3.3.1. ThreadLocalMap 的构造方法](#331-threadlocalmap-的构造方法)
    - [3.3.2. ThreadLocalMap 的 set 方法](#332-threadlocalmap-的-set-方法)

<!-- /TOC -->

# 1. ThreadLocal

ThreadLocal是一个线程内部的数据存储类，可以在指定线程中存储数据，且只有在该指定线程中才可以获取存储数据。

- ThreadLocal的静态内部类ThreadLocalMap为每个Thread都维护了一个数组 table。
- ThreadLocal确定了一个数组下标，而这个下标就是value存储的对应位置。

## 1.1. 作用

- 线程隔离：提供线程内的局部变量，不同的线程之间不会相互干扰，这种变量在线程的生命周期内起作用。
- 传递数据：减少同一个线程内多个函数或组件之间一些公共变量传递的复杂度。

## 1.2. 使用场景

**某个数据是以线程为作用域且不同线程具有不同的 Lopper**

如果不采取 ThreadLocal，那么系统就必须提供一个全局的哈希表来 Handler 查找指定线程的 Lopper，这样一来就必须提供一个类似于 LooperManager 的类。

**复杂逻辑下的对象传递（如监听器的传递）**

有时一个线程中的任务过于复杂，可能表现为函数调用栈比较深以及代码入口的多样性，这时又要监听器能够贯穿整个线程的执行过程。

如果采用 ThreadLocal 可以让监听器作为线程内的全局对象而存在，在线程内部只要通过 get 方法就可以获取监听器。

如果不采取 ThreadLocal，就只能采用另外两种办法：

- 讲监听器作为参数的形式在函数调用栈中传递：函数调用栈越深，越容易混乱。
- 将监听器作为静态变量供线程访问：不具有可扩展性，有几个线程在调用，就要提供几个静态监听器对象。

## 1.3. 示例代码

```java
// 创建 Boolean 类型的 ThreadLocal 对象
ThreadLocal<Boolean> mBooleanThread = new ThreadLocal<Boolean>();
mBooleanThread.set(true);  // 主线程中设置为 true
mBooleanThread.get();      // 主线程中获取为 true

new Thread("Thread #1") {
    @Override
    public void run() {
        mBooleanThread.set(false);  // 子线程1中设置为 false
        mBooleanThread.get();       // 子线程1中获取为 false
    }
}.start();

new Thread("Thread #2") {
    @Override
    public void run() {             // 子线程2中不去设置
        mBooleanThread.get();       // 子线程2中获取为 null
    }
}.start();
```

从 ThreadLocal 的 `set()` 和 `get()` 方法可以看出，他们所操作的对象都是当前线程的 localValues 对象和 table 数组，因此在不同线程中访问同一个 ThreadLocal 的 `set()` 和 `get()` 方法，它们对 ThreadLocal 所做的读写操作仅限于各自内部，这就是为什么 ThreadLocal 可以在多个线程找那个互不干扰的存储和修改数据。

---

# 2. ThreadLocal 原理解析

## 2.1. 内部设计

### 2.1.1. 早期方案

每个 ThreadLocal 都创建一个 ThreadLocalMap，用 Thread 作为 Map 的key，要存储的局部变量作为 Map 的 value。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201128-211844-3331972f136736421ffe0bb709f5ec13.png)

### 2.1.2. JAVA 8 方案

每个 Thread 维护一个 ThreadLocalMap，用 ThreadLocal 实例本身 Map 的 key，要存储的局部变量作为 Map 的 value。

- 每个 Thread 线程内部都有一 个Map（ThreadLocalMap）
- Map 里面存储 ThreadLocal 对象（ key）和线程的变量副本（value）
- Thread 内部的 Map 是由 ThreadLocal 维护的，由 ThreadLocal 负责向 map 获取和设置线程的变量值。
- 对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201128-211905-ef79582085e6800d00aa1c3be89b075b.png)

**【优点】**

- 每个Map所存储的元素数量变少了。
- 当Thread销毁时，ThreadLocalMap也被销毁，减少内存。

## 2.2. ThreadLocal 核心方法源码

| **方法声明**                | **描述**                     |
| -------------------------- | ---------------------------- |
| protected T initialValue() | 返回当前线程局部变量的初始值 |
| public void set(T value)   | 设置当前线程绑定的局部变量   |
| public T get()             | 获取当前线程绑定的局部变量   |
| public void remove()       | 移除当前线程绑定的局部变量   |

### 2.2.1. set 方法

先获取当前线程的 ThreadLocalMap 变量，如果存在则设置值，不存在则创建并设置值。

```java
// 设置当前线程绑定的局部变量
public void set(T value) {
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取此线程对象所维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // map不为空，则设置或更新值
        map.set(this, value);
    } else {
        // map 为空，则为线程t创建一个ThreadLocalMap对象，并把value存放其中
        createMap(t, value);
    }
}

// 获取线程所维护的ThreadLocalMap对象
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

// 为线程创建一个ThreadLocalMap对象，并赋予初始值
void createMap(Thread t, T firstValue) {
    // 这里的this是调用此方法的ThreadLocal对象
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

### 2.2.2. get 方法

先获取当前线程的 ThreadLocalMap 变量，如果存在则返回值，不存在则创建并返回初始值 `null`。

```java
// 获取当前线程绑定的局部变量
public T get() {
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取此线程对象所维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 不为空，则获取值。WeakReference
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            // 强转并返回value
            T result = (T)e.value;
            return result;
        }
    }
    // map不存在 或 map 里面没有与当前ThreadLocal关联的值
    return setInitialValue();
}

// 设置初始值
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // map不为空，则设置或更新值
        map.set(this, value);
    } else {
        // map为空，则为线程t创建一个ThreadLocalMap对象，并把value存放其中
        createMap(t, value);
    }
    if (this instanceof TerminatingThreadLocal) {
        TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
    }
    return value;
}

// 返回初始值
// 延迟调用，在调用set方法之前先调用get方法时才会执行。
protected T initialValue() {
    // 如果想返回除null之外的其他初始值，可以重写此方法
    return null;
}
```

### 2.2.3. remove方法

先获取当前线程的 ThreadLocalMap 变量，如果存在则移除。

```java
// 移除当前线程绑定的局部变量
public void remove() {
    // 获取当前线程对象所维护的ThreadLocalMap对象
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null) {
        // map不为空，则移除对应的实体
        m.remove(this);
    }
}
```

---

# 3. ThreadLocalMap 源码分析

ThreadLocalMap 是 ThreadLocal 的静态内部类，并没有实现 Map 接口。

## 3.1. ThreadLocalMap 类的基本结构

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201128-212112-18fc8ee5e58f6156ecdc97306f645fec.png)

### 3.1.1. 成员变量

```java
// 初始容量，必须是2的幂
private static final int INITIAL_CAPACITY = 16;

// 用于存放数据的table，长度必须是2的幂
private Entry[] table;

// 数组里面元素的个数，用于判断table的当前使用量是否超过阈值
private int size = 0;

// 进行扩容的阈值，当使用量大于它是就要进行扩容
private int threshold; // Default to 0
```

### 3.1.2. 存储元素

```java
// 继承自WeakReference，将ThreadLocal对象的生命周期与线程的生命周期解绑
// 如果key为null（entry.get() == null）则表示key不再被引用了，此时entry也可以从table中清除掉
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    // 只能使用ThreadLocal作为key，来存储K-V结构的数据
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

## 3.2. ThreadLocal 内存泄漏

### 3.2.1. 当key是强引用

ThreadLocalMap 中的 key 使用了强引用，会导致 threadLocal 和 value 出现内存泄漏。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201128-212213-b57d492102f19a517fffcd6d93d43d90.png)

- 假设在业务代码中使完 ThreadLocal，threadLocalRef被回收了。

- 由于 threadLocalMap 的 Entry 强引用了 threadLocal，造成 threadLocal 无法被回收。
- 在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，始终有引用链 threadRef -> currentThread -> threadLocalMap -> entry，Entry就不会被回收，导致Entry内存泄漏（threadLocal 和 value 同时出现内存泄漏）。

### 3.2.2. 当key是弱引用

ThreadLocalMap 中的 key 使用了弱引用，会导致 value 出现内存泄漏。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20201128-212232-45299201875de58bb0a90772e3226652.png)

- 假设在业务代码中使完 ThreadLocal，threadLocalRef被回收了。
- 由于 ThreadLocalMap 只持有 ThreadLocal 的弱引用，没有任何强引用指向 threadlocal 实例，所以 threadlocal 就可以顺利被gc回收，此时 Entry 中的 key=null。
- 在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，也存在有强引用链 threadRef -> currentThread -> threadLocalMap -> entry -> value，value不会被回收，而这块 value 永远不会被访问到了，导致 value 内存泄漏。

### 3.2.3. 导致内存泄漏的原因

- 没有手动删除相应的Entry对象
- 当前线程依然在运行

**【解决办法】**

- 使用完 ThreadLocal，调用其 remove 方法删除对应的 Entry。
- 使用完 ThreadLocal，当前 Thread 也随之运行结束。（不好控制，线程池中的核心线程不会销毁）

## 3.3. 解决Hash冲突

### 3.3.1. ThreadLocalMap 的构造方法

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    // 初始化table数组，长度为16
    table = new Entry[INITIAL_CAPACITY];
    // 计算索引【重点分析】
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    // 在相应索引位置存放对应的值
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    // 设置阈值
    setThreshold(INITIAL_CAPACITY);
}
```

**【firstKey.threadLocalHashCode】**

```java
// 这个值跟斐波那契数列（黄金分割数）有关。
// 目的是让哈希码能均匀的分布在2的n次方的数组里，即Entry[]，可以尽量避免hash冲突。
private static final int HASH_INCREMENT = 0x61c88647;

private final int threadLocalHashCode = nextHashCode();

private static AtomicInteger nextHashCode = new AtomicInteger();

private static int nextHashCode() {
    // 每次获取值时，把当前值加上HASH_INCREMENT并返回
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

**【& (INITIAL_CAPACITY - 1)】**

计算 hash 时，采用 `hashCode & (size - 1)` 的算法，是取模运算 `hashCode % size` 的高效实现。

正因为这种算法，要求 `size` 必须是2的整次幂，保证在索引不越界的前提下，使得 hash 发生冲突的次数减小。

### 3.3.2. ThreadLocalMap 的 set 方法

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    // 计算索引【重点分析】
    int i = key.threadLocalHashCode & (len-1);

    // 使用线性探测法查找元素【重点分析】
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // 对应的key存在，则直接覆盖旧的值
        if (k == key) {
            e.value = value;
            return;
        }
        // key为null，值不为null，说明之前的ThreadLocal对象已经被回收了
        if (k == null) {
            // 使用新的元素替换旧的元素
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 探测完毕都没找到，则在空元素位置创建一个新的Entry
    tab[i] = new Entry(key, value);
    // 叠加一个数量
    int sz = ++size;
    // cleanSomeSlots用于清除那些e.get())==null的元素
    // 这种数据key关联的对象已经被回收，所以这个Entry(table[index])可以被置null.
    // 如果没有清除任何entry，并且当前使用量达到了负载因子所定义(长度的2/3)，那么进行
    // rehash (执行一次全表的扫描清理工作) 
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

// 获取环形数组的下一个索引
private static int nextIndex(int i, int len) {
    // 如果当前索引+1之后越界了，则返回索引0
    return ((i + 1 < len) ? i + 1 : 0);
}
```

**【执行流程】**

1. 根据`key`计算出索引`i`，然后查找`i`位置上的Entry。
2. 如果Entry存在并且`key`等于传入的`key`，则直接给这个Entry赋新的`value`值。
3. 如果Entry存在但是`key`等于`null`，则调用`replaceStaleEntry`来更换这个`key`为`null`的Entry。
4. 循环检测，直到遇到为`null`的地方，此时如果循环还没有`return`，就在这个`null`的位置新建一个Entry，并且插入，同时`size`增加1。
5. 最后调用`cleanSomeSlots`，清理`key`为`null`的Entry。
6. 返回是否清理了Entry，接下来再判断`size`是否达到了`rehash`的条件（size≥thresgold），达到的话就会调用`rehash`函数执行一次全表的扫描清理。

**【线性探测法】**

用来解决哈希冲突。

- 依次探测下一个地址，直到有空的地址后插入，若整个空间都找不到空余的地址，则产生溢出。
- 举例，假设当前table长度为16，计算出来key的hash值为14，如果`table[14]`上已经有值，且其key与当前key不一致，那么就发生了hash冲突，这时将14加1得到15，取`table[15]`进行判断，如果还冲突会回到0，取table[0]，以此类推直到可以插入为止。
- 按照上面的描述，可以把`Entry[] table`看成一个环形数组。