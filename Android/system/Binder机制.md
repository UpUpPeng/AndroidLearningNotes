<!-- TOC -->

- [1. 传统的Linux进程间通信](#1-传统的linux进程间通信)
  - [1.1. 基本原理](#11-基本原理)
  - [1.2. 管道和消息队列的原理](#12-管道和消息队列的原理)
- [2. Binder原理](#2-binder原理)
  - [2.1. Binder 是什么](#21-binder-是什么)
  - [2.2. 动态内核可加载模块（LKM）](#22-动态内核可加载模块lkm)
  - [2.3. 内存映射（mmap）](#23-内存映射mmap)
  - [2.4. Binder 数据流转原理](#24-binder-数据流转原理)
- [3. Binder 通信模型](#3-binder-通信模型)
  - [3.1. Client / Server / ServiceManager / Binder Dirver](#31-client--server--servicemanager--binder-dirver)
  - [3.2. Binder 通信过程](#32-binder-通信过程)
- [4. Binder 原理步骤](#4-binder-原理步骤)

<!-- /TOC -->

# 1. 传统的Linux进程间通信

## 1.1. 基本原理

![](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-112910-3fc5f029675e956ef5cac8d9aaf14fc0.png)

【**进程隔离**】

进程与进程间内存不共享；如果要进行数据交换，就得采用特殊的通信机制：进程间通信（IPC）。

【**进程空间划分**】

为了保护用户进程不能直接操作内核，保证内核的安全，操作系统从逻辑上将虚拟空间划分为用户空间（User Space）和内核空间（Kernel Space）。

**【系统调用】**

用户空间需要访问内核空间的资源，比如文件操作、访问网络等等。就需要借助唯一的方式系统调用来实现。保证所有的资源访问都是在内核的控制下进行的，提升了系统安全性和稳定性。

- 当进程执行系统调用而陷入内核代码中执行时，处于内核运行态（内核态）。
- 当进程在执行用户自己的代码的时，处于用户运行态（用户态）。

系统调用主要通过如下两个函数来实现：

```java
copy_from_user() // 将数据从用户空间拷贝到内核空间
copy_to_user()   // 将数据从内核空间拷贝到用户空间
```

## 1.2. 管道和消息队列的原理

![](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-113153-004b7ff937dd39a0824b564225d10c5e.png)

**【流程步骤】**

1. 发送进程将要发送的数据存放在内存缓存区中，通过系统调用进入内核态。
2. 内核程序在内核空间开辟一块内核缓存区，调用 copy_from_user() 函数将数据从用户空间的内存缓存区拷贝到内核空间的内核缓存区中。
3. 接收进程在接收数据时在自己的用户空间开辟一块内存缓存区，然后内核程序调用 copy_to_user() 函数将数据从内核缓存区拷贝到接收进程的内存缓存区。

**【缺点】**

1. 性能低下，一次数据传递需要**2次**数据拷贝：内存缓存区 --> 内核缓存区 --> 内存缓存区。
2. 接收进程并不知道接收数据的大小，因此只能开辟尽可能大的内存空间（浪费空间）或者先调用 API 接收消息头来获取消息体的大小（浪费时间）。

------

# 2. Binder原理

## 2.1. Binder 是什么

- 从进程间通信的角度看，Binder 是一种进程间通信的机制。

- 从 Server 进程的角度看，Binder 指的是 Server 中的 Binder 实体对象。
- 从 Client 进程的角度看，Binder 指的是 Binder 实体对象的一个远程代理。
- 从传输过程的角度看，Binder 是一个可以跨进程传输的对象，Binder 驱动会对这个跨越进程边界的对象对一点点特殊处理，自动完成代理对象和本地对象之间的转换。

## 2.2. 动态内核可加载模块（LKM）

- 内核模块是具有独立功能的程序，它可以被单独编译，但不能独立运行。
- 内核模块在运行时被链接到内核作为内核的一部分运行。
- 可以动态添加一个内核模块运行在内核空间。

由于 Binder 不是 Liunx 系统内核的一部分，因此 Android 系统中，负责各个用户进程通过 Binder 实现通信的内核模块（ Binder 驱动，Binder Dirver）来作为桥梁来实现通信。

## 2.3. 内存映射（mmap）

- 将用户空间的一块内存区域映射到内核空间。
- 映射关系建立后，用户空间对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这块内存区域的修改也可以直接反应到用户空间。
- 减少数据拷贝次数（只需要1次），实现用户空间和内核空间的高效互动。

## 2.4. Binder 数据流转原理

![](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-113348-8491d03e4d0e049a6921ca01e90d7c4c.png)

1. Binder 驱动在内核空间创建一个数据接收缓存区。
2. 建立内核缓存区和数据接收缓存区的映射关系，以及数据接收缓存区和接收进程用户空间地址的映射关系。
3. 发送方进程通过系统调用 copy_from_user() 将数据 copy 到内核缓存区。
4. 由于内核缓存区&接收进程的用户空间地址存在映射关系（同时映射 Binder 创建的接收缓存区中），故相当于也发送到了接收进程的用户空间地址，即实现了跨进程通信。

------

# 3. Binder 通信模型

一次完整的 IPC 至少包含两个进程，客户端进程（Client）和服务端进程（Server）。

## 3.1. Client / Server / ServiceManager / Binder Dirver

![](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-113553-5e4725e75b7b00d52b73fd6d8e2c5bcd.png)

**【关系】**

- Client、Server、Service Manager 运行在用户空间，Binder 驱动运行在内核空间。
- Client、Server、ServiceManager 通过系统调用 open、mmap 和 ioctl 来访问设备文件 /dev/binder 来间接的实现跨进程通信。
- Service Manager 、Binder 驱动由系统提供，Client、Server 由应用程序实现。

**【作用】**

- Binder Dirver：类似路由器，负责 Binder 通信的建立，Binder 的传递，Binder 引用计数管理，数据包的传递和交互等底层支持。
- ServiceManager：类似DNS，将字符形式的 Binder 名字转化成 Client 中对该 Binder 的引用，使得 Client 能够通过 Binder 的名字获得对 Binder 实体的引用。
- Client：获得实名 Binder 的引用。

## 3.2. Binder 通信过程

![](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-113938-78cca8812b7022a99b359bb334783dca.png)

**1. 系统创建ServiceManager：**

ServiceManager 进程要想成为 ServiceManager ，就必须先通过 Binder 驱动向系统注册，但是此时整个 IPC 机制还没建立，因此 ServiceManager 进程会使用 BINDER_SET_CONTEXT_MGR 命令，Binder 驱动就会自动为它创建 Binder 实体（ 0 号引用），以后的 Server 进程必须通过这个 0 号引用和 ServiceManager 的 Binder 通信。

**2. 注册 Server 进程：**

Server 进程通过 Binder 驱动向 ServiceManager 中注册 Binder（Server 中的 Binder 实体），表明可以对外提供服务。 Binder 驱动为这个 Binder 创建位于内核中的实体节点以及 ServiceManager 对实体的引用，将名字以及新建的引用打包传给 ServiceManager，ServiceManger 将其填入查找表。

**3. Client 进程建立联系：**

Client 进程通过 Server 进程的名字，在 Binder 驱动的帮助下从 ServiceManager 中获取到对应 Binder 实体的引用，通过这个引用就能实现和 Server 进程的通信。

**4. 数据通信：**

Server 进程通过系统调用 copy_from_user() 将数据 copy 到内核缓存区，Client 进程通过两层内存映射，可以直接使用内核缓存区中的数据。

---

# 4. Binder 原理步骤

![img](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-114028-93aae46f68347846f1cf2ee2b1d107f3.png)

![img](https://raw.githubusercontent.com/ren-p/AndroidLearningNotes/main/img/20201129-114041-fbf2b12da0d2b1ab5291ed73cf6a823f.png)