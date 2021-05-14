# 1. 系统启动流程概括

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210317-102619-926f3ab3b445eb306aca5b47758c5fef.png)

启动的主要流程：

1. **启动电源以及系统启动：** 当电源按下时，引导芯片代码从预定义的地方（固化在 ROM）开始执行。加载引导程序 BootLoader 到 RAM，然后执行。
2. **引导程序 Bootloader：** 引导程序 BootLoader 是在 Android 操作系统开始运行前的一个小程序，它的主要作用是把操作系统拉起来并运行。
3. **Linux 内核启动：** 当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。当内核完成系统设置时，它首先在系统文件中寻找 `init.rc` 文件，并启动 init 进程 。 
4. **init 进程启动：** 初始化和启动属性服务，井且启动 Zygote 进程。
5. **Zygote 进程启动：** 创建 Java 虚拟机并为 Java 虚拟机注册 JNI 方法，创建服务器端 Socket，启动 SystemServer 进程。
6. **SystemServer 进程启动：** 启动 Binder 线程池和 SystemServiceManager，并且启动各种系统服务。
7. **Launcher 启动：** 被 SystemServer 进程启动的 AMS 会启动 Launcher，Launcher 启动后会将已安装应用的快捷图标显示到界面上。

---

<br/>

# 2. init 进程启动

init 进程是 Android 系统中用户空间的第一个进程，进程号为 1，是 Android 系统启动流程中一个关键的步骤。主要做了以下三件事情：

- 创建和挂载启动所需的文件目录。
- 初始化和启动属性服务。
- 解析 `init.rc` 配置文件并启动 Zygote 进程。

## 2.1. init 进程的入口函数

在 Linux 内核加载完成后，它首先在系统文件中寻找 `init.rc` 文件（`/system/core/rootdir/init.rc`），并启动 init 进程，然后查看 init 进程的入口 `main` 函数来读取 `init.rc` 中的相关配置，从而来启动其他相关进程以及其他操作。

Android 9 及以前，main 函数在 `/system/core/init/init.cpp` 文件中，以下源码基于 Android 9.0.0：

```c++
int main(int argc, char** argv) {
    /* ... */

    bool is_first_stage = (getenv("INIT_SECOND_STAGE") == nullptr);

    if (is_first_stage) {
        boot_clock::time_point start_time = boot_clock::now();

        // 清理umask.
        umask(0);

        clearenv();
        setenv("PATH", _PATH_DEFPATH, 1);
        // 创建和挂载启动所需要的文件目录
        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
        mkdir("/dev/pts", 0755);
        mkdir("/dev/socket", 0755);
        mount("devpts", "/dev/pts", "devpts", 0, NULL);
        #define MAKE_STR(x) __STRING(x)
        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
        // Don't expose the raw commandline to unprivileged processes.
        chmod("/proc/cmdline", 0440);
        gid_t groups[] = { AID_READPROC };
        setgroups(arraysize(groups), groups);
        mount("sysfs", "/sys", "sysfs", 0, NULL);
        mount("selinuxfs", "/sys/fs/selinux", "selinuxfs", 0, NULL);

        mknod("/dev/kmsg", S_IFCHR | 0600, makedev(1, 11));

        if constexpr (WORLD_WRITABLE_KMSG) {
            mknod("/dev/kmsg_debug", S_IFCHR | 0622, makedev(1, 11));
        }

        mknod("/dev/random", S_IFCHR | 0666, makedev(1, 8));
        mknod("/dev/urandom", S_IFCHR | 0666, makedev(1, 9));

        mount("tmpfs", "/mnt", "tmpfs", MS_NOEXEC | MS_NOSUID | MS_NODEV,
              "mode=0755,uid=0,gid=1000");

        mkdir("/mnt/vendor", 0755);

        // 初始化Kernel的Log，这样就可以从外界获取Kernel的日志
        InitKernelLogging(argv);
        /* ... */
    }
    /* ... */
	// 对属性服务进行初始化
    property_init();
    /* ... */
	// 创建 epoll 句柄
    epoll_fd = epoll_create1(EPOLL_CLOEXEC);
    /* ... */
	// 用于设置子进程信号处理函数，如果子进程(Zygote进程)异常退出，init进程会调用该函数中设定的信号处理函数来进行处理
    sigchld_handler_init();

    /* ... */
	// 导入默认的环境变量
    property_load_boot_defaults();
    export_oem_lock_status();
    // 启动属性服务
    start_property_service();
    set_usb_controller();

    const BuiltinFunctionMap function_map;
    Action::set_function_map(&function_map);

    subcontexts = InitializeSubcontexts();

    ActionManager& am = ActionManager::GetInstance();
    ServiceList& sm = ServiceList::GetInstance();

    // 加载启动脚本，具体内容看下面的函数👇
    LoadBootScripts(am, sm);
    
	if (false) DumpState();
    /* ... */
    while (true) {
        /* ... */
        if (!(waiting_for_prop || Service::is_exec_service_running())) {
            // 内部遍历执行每个action中携带的command对应的执行函数
            am.ExecuteOneCommand();
        }
        if (!(waiting_for_prop || Service::is_exec_service_running())) {
            if (!shutting_down) {
                // 重启死去的进程
                auto next_process_restart_time = RestartProcesses();
                /* ... */
            }
			/* ... */
        }
		/* ... */
    }
    
    return 0;
}

static void LoadBootScripts(ActionManager& action_manager, ServiceList& service_list) {
    Parser parser = CreateParser(action_manager, service_list);

    std::string bootscript = GetProperty("ro.boot.init_rc", "");
    if (bootscript.empty()) {
        // 解析init.rc配置文件
        parser.ParseConfig("/system/etc/init/hw/init.rc");
        if (!parser.ParseConfig("/system/etc/init")) {
            late_import_paths.emplace_back("/system/etc/init");
        }
        parser.ParseConfig("/system_ext/etc/init");
        if (!parser.ParseConfig("/vendor/etc/init")) {
            late_import_paths.emplace_back("/vendor/etc/init");
        }
        if (!parser.ParseConfig("/odm/etc/init")) {
            late_import_paths.emplace_back("/odm/etc/init");
        }
        if (!parser.ParseConfig("/product/etc/init")) {
            late_import_paths.emplace_back("/product/etc/init");
        }
    } else {
        parser.ParseConfig(bootscript);
    }
}
```

Android 10 及以后，main 函数在 `/system/core/init/main.cpp` 文件中，以下源码基于 Android 10.0.0：

```c++
// argc表示argv中有多少个参数
// 参数argv可能有四种情况，进入不同的入口
int main(int argc, char** argv) {
#if __has_feature(address_sanitizer)
    __asan_set_error_report_callback(AsanReportCallback);
#endif
	// 参数中有ueventd，进入ueventd_main，ueventd主要是负责设备节点的创建、权限设定等一些列工作
    if (!strcmp(basename(argv[0]), "ueventd")) {
        return ueventd_main(argc, argv);
    }

    if (argc > 1) {
        // 参数中有subcontext，进入InitLogging和SubcontextMain，初始化日志系统
        if (!strcmp(argv[1], "subcontext")) {
            android::base::InitLogging(argv, &android::base::KernelLogger);
            const BuiltinFunctionMap function_map;

            return SubcontextMain(argc, argv, &function_map);
        }
		// 参数中有selinux_setup，进入SetupSelinux，启动Selinux安全策略
        if (!strcmp(argv[1], "selinux_setup")) {
            return SetupSelinux(argv);
        }
		// 参数中有second_stage，进入SecondStageMain，启动init进程第二阶段
        if (!strcmp(argv[1], "second_stage")) {
            return SecondStageMain(argc, argv);
        }
    }
	// 启动init进程第一阶段（函数定义在/system/core/init/init.cpp中）
    return FirstStageMain(argc, argv);
}
```

综上，main 函数主要做了以下关键步骤：

- **创建和挂载启动所需要的文件目录：** 挂载 `tmpfs`、`devpts`、`proc`、`sysfs` 和 `selinuxfs` 5种文件系统，这些都是系统运行时目录。
- **property_init()：** 对属性进行初始化。
- **signal_handler_init()：** 设置子进程信号处理函数，用于防止 init 进程的子进程（Zygote）成为**僵尸进程**。
- **start_property_service()：** 启动属性服务。
- **parser.ParseConfig("/system/etc/init/hw/init.rc")：** 解析 `init.rc` 文件。
- **restart_processes()：** 重启死去的进程。

**【僵尸进程】**

父进程使用 fork 创建子进程，如果子进程终止后，父进程并不知道，而且在系统进程表中仍然保留了它的信息（如进程号、退出状态、运行时间...），则已终止的子进程就是僵尸进程。

系统进程表如果被僵尸进程耗尽的话，系统就可能无法创建新的进程了。

假设 init 进程的子进程 Zygote 终止了，`signal_handler_init()` 内部会找到 Zygote 进程井移除全部信息，再重启 Zygote 服务的启动脚本中带有 `onrestart` 选项的服务，并调用 `restart_processes()` 重启 Zygote 进程。

## 2.2. 解析 init.rc 文件

 `/system/core/rootdir/init.rc` 是由 Android 初始化语言（Android Init Language）编写的配置文件，主要包含5种类型语句：Action，Command，Service，Option 和 Import。

```c
/* ... */
// 按住系统电源键开机时，会触发此动作
on property:sys.boot_from_charger_mode=1
    class_stop charger
    trigger late-init
/* ... */
// 挂载文件系统并启动核心系统服务。
on late-init
    /* .. */
    // 基于文件加密的设备启动zygote进程
    trigger zygote-start
	/* .. */
```

其中，重要的步骤是启动 zygote 进程。

`/system/core/rootdir/init.zygoteXX.rc` 文件是 Zygote 的启动脚本。

```shell
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    socket usap_pool_primary stream 660 root system
    onrestart exec_background - system system -- /system/bin/vdc volume abort_fuse
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```

Service 用于通知 `init` 进程创建名为 `zygote` 的进程，执行程序路径为 `/system/bin/app_process64`，后面的是要传给 `app_process64` 的参数，且 Zygote 的 `classname` 为 `main`。如果 `audioserver`、`cameraserver`、`media` 等进程终止了，就需要进行 restart。

---

<br/>

# 3. Zygote 进程启动

Zygote 进程称为孵化器，DVM 和 ART 是由它创建的，APP进程和系统的 SystemServer 进程是由它 fock（复制进程） 创建的，因此可以在内部获取一个 DVM 或者 ART 的实例副本。

> Zygote 进程的名称并不是“zygote”，而是“app_process”，这个名称是在 Android.mk 中定义的。Zygote 进程启动后，Linux 系统下的 pctrl 系统会调用 app_process，将其名称换成了“zygote”。

> 不同版本，代码略有差别，总体流程差别不大，以下代码基于 Android 8.0 源码

## 3.1. 启动过程

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210514-150228-dee721d7e2847898c75a3d139b2ed2a3.png)

### 3.1.1. app_main.cpp

> frameworks/base/cmds/app_process/app_main.cpp

`app_main.cpp` 的 `main` 函数的重要流程

```c++
int main(int argc, char* const argv[])
{
    /* ... */
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    /* ... */
    ++i;	// 跳过第一个参数（父目录）
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            // 如果main函数运行在Zygote进程中，则将zygote设置为true
            zygote = true;
            // 设置进程的名字 
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            // 如果main函数运行在SystemServer进程中，则将startSystemServer设置为true
            startSystemServer = true;
        } 
        /* ... */
    }
    /* ... */
    if (zygote) {
        // 如果运行在Zygote进程中，则调用AppRuntime的start函数
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}
```

**重要流程概括：**

- 判断当前 `main` 函数是运行在 Zygote 进程还是 SystemServer 进程。
  - Zygote 进程通过 fock 自身来创建子进程，因此 Zygote 进程和它的子进程都可以进入 `app_main.cpp` 的 `main` 函数， 所以 `main` 函数中必须区分当前运行在哪个进程。
- 如果运行在 Zygote 进程中，则调用 AppRuntime 的 `start` 函数来启动 Java 虚拟机。

### 3.1.2. AndroidRuntime.cpp

> frameworks/base/core/jni/AndroidRuntime.cpp

`AndroidRuntime.cpp` 的 `start` 函数的重要流程

```c++
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    /* ... */
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    // 创建Java虚拟机
    if (startVm(&mJavaVM, &env, zygote, primary_zygote) != 0) {
        return;
    }
    // 对虚拟机进行初始化  
    onVmCreated(env);

    // 为Java虚拟机注册JNI方法
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }

    /* ... */
    // 从 app_main的main函数传递过来的className为com.android.internal.os.ZygoteInit
    classNameStr = env->NewStringUTF(className);
    assert(classNameStr != NULL);
    env->SetObjectArrayElement(strArray, 0, classNameStr);

    for (size_t i = 0; i < options.size(); ++i) {
        jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
        assert(optionsStr != NULL);
        env->SetObjectArrayElement(strArray, i + 1, optionsStr);
    }

    // 启动虚拟机，该线程成为虚拟机的主线程，直到虚拟机退出后才返回。
    // 将classNarne的“.”替换为“/”
    char* slashClassName = toSlashClassName(className != NULL ? className : "");
    // 查找ZygoteInit这个Java类
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
    } else {
        // 找到ZygoteInit的main方法
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V");
        if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
            /* keep going */
        } else {
            // 通过JNI调用ZygoteInit的main方法
            env->CallStaticVoidMethod(startClass, startMeth, strArray);
            /* ... */
        }
    }
    /* ... */
}
```

**重要流程概括：**

- 调用 `startVm` 函数来创建 Java 虚拟机。
- 调用 `startReg` 函数为 Java 虚拟机注册 JNI 方法。
- 将传递过来的参数 `className` 的值 `com.android.internal.os.ZygoteInit` 中的`.`替换为`/`，即 `com/android/internal/os/ZygoteInit`，并赋值给 `slashClassName` 。
- 根据 `slashClassName` 去查找 `Zygotelnit` 类。
- 如果找到了，则通过 JNI 调用 `ZygoteInit` 的 `main` 方法。

⭐️ 在此之前，一直在 native 层运行，没有任何代码进入 Java 框架层，也就是说是 Zygote 开启了 Java 框架层的运行。

### 3.1.3. ZygoteInit.java

> frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

`ZygoteInit.java` 的 `main` 方法和 `startSystemServer` 方法的重要流程

不同版本，代码略有差别：

以下是 **Android 8.0** 中的 `ZygoteInit` 类。

```java
public static void main(String argv[]) {
    // 创建ZygoteServer对象
    ZygoteServer zygoteServer = new ZygoteServer();
    /* ... */
    try {
        /* ... */
        String socketName = "zygote";
        /* ... */
        // 创建一个Server端的Socket，并传入socketName
        zygoteServer.registerServerSocket(socketName);
        if (!enableLazyPreload) {
            bootTimingsTraceLog.traceBegin("ZygotePreload");
            EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_START, SystemClock.uptimeMillis());
            // 预加载类和资源
            preload(bootTimingsTraceLog);
            EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_END, SystemClock.uptimeMillis());
            bootTimingsTraceLog.traceEnd();
        } else {
            Zygote.resetNicePriority();
        }
		/* ... */
        if (startSystemServer) {
            // 启动SystemServer进程
            startSystemServer(abiList, socketName, zygoteServer);
        }
        Log.i(TAG, "Accepting command socket connections");
        // 等待AMS创建新应用程序进程的请求
        zygoteServer.runSelectLoop(abiList);
        zygoteServer.closeServerSocket();
    } catch (Zygote.MethodAndArgsCaller caller) {
        caller.run();
    } catch (Throwable ex) {
        Log.e(TAG, "System zygote died with exception", ex);
        zygoteServer.closeServerSocket();
        throw ex;
    }
}

private static boolean startSystemServer(String abiList, String socketName, ZygoteServer zygoteServer)
        throws Zygote.MethodAndArgsCaller, RuntimeException {
    /* ... */
    // 创建args数组，保存启动SystemServer的启动参数
    String args[] = {
        "--setuid=1000", 	// 用户id
        "--setgid=1000",	// 用户组id
        // 进程拥有的权限
        "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,1032,3001,3002,3003,3006,3007,3009,3010",
        "--capabilities=" + capabilities + "," + capabilities,
        "--nice-name=system_server",	// 进程名
        "--runtime-args",
        // 启动的类名
        "com.android.server.SystemServer",
    };
    ZygoteConnection.Arguments parsedArgs = null;

    int pid;

    try {
        // 将args数组封装成Arguments对象
        parsedArgs = new ZygoteConnection.Arguments(args);
        ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
        ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

        // 创建一个子进程，即SystemServer进程
        pid = Zygote.forkSystemServer(
                parsedArgs.uid, parsedArgs.gid,
                parsedArgs.gids,
                parsedArgs.debugFlags,
                null,
                parsedArgs.permittedCapabilities,
                parsedArgs.effectiveCapabilities);
    } catch (IllegalArgumentException ex) {
        throw new RuntimeException(ex);
    }

    // 当前代码逻辑运行在子进程中
    if (pid == 0) {
        if (hasSecondZygote(abiList)) {
            waitForSecondaryZygote(socketName);
        }
		// 关闭Zygote进程创建的Socket
        zygoteServer.closeServerSocket();
        // 处理SystemServer进程
        handleSystemServerProcess(parsedArgs);
    }

    return true;
}
```

**重要流程概括：**

- 通过 `registerServerSocket` 方法创建一个 Server 端名为 `zygote` 的 Socket 对象。
- 预加载公共类和公共资源，而这些资源可以通过 fork 的方式，共享给它的全部子进程，加载又分为几个部分：
  - `preloadClasses(); ` 预加载 Classes。
  - `preloadResources();` 预加载 Resources。
  - `preloadOpenGL();` 预加载 openGL。
  - `preloadSharedLibraries();` 预加载共享类库。
  - `preloadTextResources();` 预加载文本资源。
  - `WebViewFactory.prepareWebViewInZygote();` 初始化 WebView，以实现子进程间内存共享。
- 启动 SystemServer 进程，系统服务也会由 SystemServer 进程启动起来。
- 调用 `ZygoteServer` 的 `runSelectLoop` 方法等待 `ActivityManagerService` 请求创建新应用程序进程。

### 3.1.4. ZygoteServer.java

> frameworks/base/core/java/com/android/internal/os/ZygoteServer.java

`ZygoteServer.java` 的 `registerServerSocket` 方法和 `runSelectLoop` 方法的重要流程

以下是 **Android 8.0** 中的 `ZygoteServer` 类。

```java
private static final String ANDROID_SOCKET_PREFIX = "ANDROID_SOCKET_";
// 服务器端Socket对象
private LocalServerSocket mServerSocket;

void registerServerSocket(String socketName) {
    if (mServerSocket == null) {
        int fileDesc;
        // 拼接完整的Socket名称，socketName其值为zygote
        final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;
        try {
            // 获取环境变量
            String env = System.getenv(fullSocketName);
            // 将环境变量的值转换为文件描述符的参数
            fileDesc = Integer.parseInt(env);
        } catch (RuntimeException ex) {
            throw new RuntimeException(fullSocketName + " unset or invalid", ex);
        }

        try {
            FileDescriptor fd = new FileDescriptor();
            // 设置文件描述符  
            fd.setInt$(fileDesc);
            // 根据文件描述符创建服务器端Socket对象
            mServerSocket = new LocalServerSocket(fd);
        } catch (IOException ex) {
            throw new RuntimeException("Error binding to local socket '" + fileDesc + "'", ex);
        }
    }
}

void runSelectLoop(String abiList) throws Zygote.MethodAndArgsCaller {
    // 文件描述符列表
    ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
    // Socket连接列表
    ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
	// 获得该Socket的文件描述符字并添加到fds中
    fds.add(mServerSocket.getFileDescriptor());
    peers.add(null);
	// 无限循环等待AMS的请求
    while (true) {
        // 将fds存储的信息转移到pollFds数组中
        StructPollfd[] pollFds = new StructPollfd[fds.size()];
        for (int i = 0; i < pollFds.length; ++i) {
            pollFds[i] = new StructPollfd();
            pollFds[i].fd = fds.get(i);
            pollFds[i].events = (short) POLLIN;
        }
        try {
            Os.poll(pollFds, -1);
        } catch (ErrnoException ex) {
            throw new RuntimeException("poll failed", ex);
        }
        for (int i = pollFds.length - 1; i >= 0; --i) {
            if ((pollFds[i].revents & POLLIN) == 0) {
                continue;
            }
            if (i == 0) {
                // 服务器端Socket与客户端连接上了，即当前Zygote进程与AMS建立了连接
                ZygoteConnection newPeer = acceptCommandPeer(abiList);
                // 添加到peers和fds中，以便可以接收到AMS发送过来的请求。
                peers.add(newPeer);
                fds.add(newPeer.getFileDesciptor());
            } else {
                // AMS向Zygote进程发送了一个创建应用进程的请求
                // 创建一个新的应用程序进程
                boolean done = peers.get(i).runOnce(this);
                if (done) {
                    // 创建成功，将这个连接从peers和fds中清除。
                    peers.remove(i);
                    fds.remove(i);
                }
            }
        }
    }
}
```

**重要流程概括：**

- `registerServerSocket` 方法：创建一个服务器端的 Socket，用于等待 AMS 请求 Zygote 进程来创建新的应用程序进程。
- `runSelectLoop` 方法：无限循环等待 AMS 的请求，一旦有请求，则调用 `ZygoteConnection` 的 `runOnce` 函数来创建一个新的应用程序进程。

## 3.2. 启动总结

- Init 进程通过 `init.zygoteXX.rc` 文件，创建 AppRuntime 并调用其 `start` 函数，启动 Zygote 进程。
- AppRuntime 创建 Java 虚拟机并为 Java 虚拟机注册 JNI方法。
- AppRuntime 通过 JNI 调用 Zygotelnit 的 `main` 方法进入 Zygote 的 Java 框架层。
- Zygotelnit 通过 ZygoteServer 的 `registerZygoteSocket` 方法创建服务器端 Socket。
- Zygotelnit 预加载公共类和公共资源。
- Zygotelnit 启动 SystemServer 进程。
- Zygotelnit 通过 ZygoteServer 的 `runSelectLoop` 方法等待 AMS 的请求来创建新的应用程序进程。

---

<br/>

# 4. SystemServer 的处理

SystemServer 进程主要用于创建系统服务，例如 AMS、WMS 和 PMS 都是由它来创建的。

> 不同版本，代码略有差别，总体流程差别不大，以下代码基于 Android 8.0 源码

## 4.1. 启动 SystemServer 进程

Zygotelnit.java 调用自身的 `startSystemServer` 方法，启动 SystemServer 进程。

> frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
private static boolean startSystemServer(String abiList, String socketName, ZygoteServer zygoteServer)
        throws Zygote.MethodAndArgsCaller, RuntimeException {
    /* ... */
    // 当前代码逻辑运行在子进程中
    if (pid == 0) {
        if (hasSecondZygote(abiList)) {
            waitForSecondaryZygote(socketName);
        }
		// 关闭Zygote进程fork得到的Socket，该Scoket对SystemServer进程没有用处
        zygoteServer.closeServerSocket();
        // 启动SystemServer进程
        handleSystemServerProcess(parsedArgs);
    }
    return true;
}
```

调用 `handleSystemServerProcess` 方法来启动 SystemServer 进程。

```java
private static void handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs)
        throws Zygote.MethodAndArgsCaller {
	/* ... */
    if (parsedArgs.invokeWith != null) {
        /* ... */
    } else {
        ClassLoader cl = null;
        if (systemServerClasspath != null) {
            // 创建了PathClassLoader
            cl = createPathClassLoader(systemServerClasspath, parsedArgs.targetSdkVersion);
			// 并把PathClassLoader设置为当前线程的ContextClassLoader
            Thread.currentThread().setContextClassLoader(cl);
        }
        // 将其余的参数传递给SystemServer。
        ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
    }
}

public static final void zygoteInit(int targetSdkVersion, String[] argv,
        ClassLoader classLoader) throws Zygote.MethodAndArgsCaller {
    /* ... */
    RuntimeInit.commonInit();
    // 使用Native代码启动Binder线程池，这样SystemServer进程就可以使用Binder与其他进程进行通信了
    ZygoteInit.nativeZygoteInit();
    // 进入SystemServer的main方法
    RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
}
```

### 4.1.1. 启动 Binder线程池

`nativeZygoteInit()` 对应 `AndroidRuntime.cpp` 中的 `com_android_internal_os_ZygoteInit_nativeZygoteInit` 函数：

> frameworks/base/core/jni/AndroidRuntime.cpp

```c++
static AndroidRuntime* gCurRuntime = NULL;
static void com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    // gCurRuntime指向AndroidRuntime的子类AppRuntime
    gCurRuntime->onZygoteInit();
}
```

AppRuntime 类在 app_main.cpp 中定义。

> frameworks/base/cmds/app_process/app_main.cpp

```c++
virtual void onZygoteInit()
{
    sp<ProcessState> proc = ProcessState::self();
    ALOGV("App process: starting thread pool.\n");
    // 启动Binder线程池
    proc->startThreadPool();
}
```

### 4.1.2. main 方法启动

`RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);` 调用了 `RuntimeInit` 类的 `applicationInit` 静态方法来启动 SystemServer 进程。

> frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
protected static void applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
        throws Zygote.MethodAndArgsCaller {
    /* ... */
    invokeStaticMain(args.startClass, args.startArgs, classLoader);
}

private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader)
        throws Zygote.MethodAndArgsCaller {
    /* ... */
        // 通过反射得到SystemServer类，className的值为com.android.server.SystemServer
        cl = Class.forName(className, true, classLoader);
    /* ... */
        //找到SystemServer的main方法
        m = cl.getMethod("main", new Class[] { String[].class });
    /* ... */
    // 抛出MethodAndArgsCaller异常，并把main方法和参数带出去
    throw new Zygote.MethodAndArgsCaller(m, argv);
}
```

捕获 `MethodAndArgsCaller` 异常的代码在 `Zygotelnit.java` 的 `main` 方法中。然后才会调用 SystemServer 的 `main` 方法。

不直接调用 `main` 方法，而采用抛出异常的方式，有两点原因：

- 这种抛出异常的处理会清除所有的设置过程需要的堆栈帧。
- 让 SystemServer 的 `main` 方法看起来像是 SystemServer 进程的入口方法。
  - 在 Zygote 启动了 SystemServer 进程后，SystemServer 进程已经做了很多的准备工作，而这些工作都是在 SystemServer的 `main` 方法调用之前做的。
  - 这使得 SystemServer 的 `main` 方法看起来不像是 SystemServer 进程的入口方法。
  - 而这种抛出异常交由 Zygotelnit.java 的 `main` 方法来处理，会让 SystemServer 的 `main` 方法看起来像是 SystemServer 进程的入口方法。

> frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
public static void main(String argv[]) {
    ZygoteServer zygoteServer = new ZygoteServer();
	/* ... */
    try {
        /* ... */
        if (startSystemServer) {
            // 启动SystemServer进程，并捕获Zygote.MethodAndArgsCaller异常
            startSystemServer(abiList, socketName, zygoteServer);
        }
        Log.i(TAG, "Accepting command socket connections");
        zygoteServer.runSelectLoop(abiList);
        zygoteServer.closeServerSocket();
    } catch (Zygote.MethodAndArgsCaller caller) {
        // SystemServer进程完成初始化，调用run方法
        caller.run();
    } catch (Throwable ex) {
        Log.e(TAG, "System zygote died with exception", ex);
        zygoteServer.closeServerSocket();
        throw ex;
    }
}
```

`Zygote.MethodAndArgsCaller` 异常类是 `Zygote` 类的静态内部类。

> frameworks/base/core/java/com/android/internal/os/Zygote.java

```java
public static class MethodAndArgsCaller extends Exception implements Runnable {
    private final Method mMethod;
    private final String[] mArgs;

    public MethodAndArgsCaller(Method method, String[] args) {
        mMethod = method;	// 这里的method就是传入的main方法
        mArgs = args;
    }

    public void run() {
        try {
            // 执行SystemServer的main方法
            mMethod.invoke(null, new Object[] { mArgs });
        } catch (IllegalAccessException ex) {
            throw new RuntimeException(ex);
        } catch (InvocationTargetException ex) {
            Throwable cause = ex.getCause();
            if (cause instanceof RuntimeException) {
                throw (RuntimeException) cause;
            } else if (cause instanceof Error) {
                throw (Error) cause;
            }
            throw new RuntimeException(ex);
        }
    }
}
```

到此为止 SystemServer 才真正开始执行 `main` 方法。

## 4.2. 解析 SystemServer 进程

进入 SystemServer 的 `main` 方法，只是调用了 `run` 方法而已。

> frameworks/base/services/java/com/android/server/SystemServer.java

```java
public static void main(String[] args) {
    // 创建SystemServer对象，并执行run方法
    new SystemServer().run();
}

private void run() {
    /* ... */
    try {
        /* ... */
        // 设定时区
        String timezoneProperty = SystemProperties.get("persist.sys.timezone");
        if (!isValidTimeZoneId(timezoneProperty)) {
            /* ... */
            SystemProperties.set("persist.sys.timezone", "GMT");
        }

        // 设定语言
        if (!SystemProperties.get("persist.sys.language").isEmpty()) {
            final String languageTag = Locale.getDefault().toLanguageTag();
            SystemProperties.set("persist.sys.locale", languageTag);
            SystemProperties.set("persist.sys.language", "");
            SystemProperties.set("persist.sys.country", "");
            SystemProperties.set("persist.sys.localevar", "");
        }

        /* ... */

        // 设置虚拟机库文件路径
        SystemProperties.set("persist.sys.dalvik.vm.lib.2", VMRuntime.getRuntime().vmLibrary());

        // 清除内存增长上限
        VMRuntime.getRuntime().clearGrowthLimit();

        // 有些设备依赖于运行时指纹生成，因此请确保在进一步启动之前已定义它。
        Build.ensureFingerprintProperty();

        // 设定环境变量访问用户条件
        Environment.setUserRequired(true);
        BaseBundle.setShouldDefuse(true);

        Parcel.setStackTraceParceling(true);

        // 设定binder服务永远运行在前台
        BinderInternal.disableBackgroundScheduling(true);

        // 设定binder线程池最大线程数
        BinderInternal.setMaxThreads(sMaxBinderThreads);

        /* ... */
        // 创建消息Looper
        Looper.prepareMainLooper();
        Looper.getMainLooper().setSlowLogThresholdMs(
                SLOW_DISPATCH_THRESHOLD_MS, SLOW_DELIVERY_THRESHOLD_MS);

        SystemServiceRegistry.sEnableServiceNotFoundWtf = true;

        // 加载动态库libandroid_servers.so
        System.loadLibrary("android_servers");
        
        /* ... */

        // 启动系统上下文
        createSystemContext();

        /* ... */

        // 创建SystemServiceManager对象
        mSystemServiceManager = new SystemServiceManager(mSystemContext);
        mSystemServiceManager.setStartInfo(mRuntimeRestart, mRuntimeStartElapsedTime, mRuntimeStartUptime);
        LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
        SystemServerInitThreadPool.start();
        /* ... */
    } finally {
        /* ... */
    }

    /* ... */

    // 启动各种服务
    try {
        t.traceBegin("StartServices");
        // 启动引导服务
        startBootstrapServices(t);
        // 启动核心服务
        startCoreServices(t);
        // 启动其他服务
        startOtherServices(t);
    } catch (Throwable ex) {
        Slog.e("System", "******************************************");
        Slog.e("System", "************ Failure starting system services", ex);
        throw ex;
    } finally {
        t.traceEnd();
    }
	/* ... */
    // 进入Loop循环，处理消息循环
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

`run` 方法中主要做了以下几件事：

- 设置时间、时区、语言。
- 设置虚拟机库文件路径 `persist.sys.dalvik.vm.lib.2`。
- 设定 `binder` 线程池最大线程数。
- 创建消息 `Looper`。
- 加载 `libandroid_servers.so` 库。
- 启动上下文。
- 创建 `SystemServiceManager` 对象，负责系统 Service 的启动。
- 启动引导服务、核心服务、其他服务，他们都是 `SystemService` 类的子类。
  - 引导服务：`ActivityManagerService`、`PowerManagerService`、`PackageManagerService` 等。
  - 核心服务：`DropBoxManagerService`、`BatteryService`、`UsageStatsService` 和 `WebViewUpdateService` 。
  - 其他服务：`CameraService`、`AlarmManagerService`、`VrManagerService` 等。
- 开启 `Looper` 消息循环。

## 4.3. 启动总结

SystemServer 进程被创建后，主要工作如下：

- 启动 Binder 线程池，这样就可以与其他进程进行通信。
- 创建 SystemServiceManager，其用于对系统的服务进行创建、启动和生命周期管理。 
- 启动各种系统服务。

