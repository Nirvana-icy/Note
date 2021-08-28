# [iOS Crash Dump Analysis](https://faisalmemon.github.io/ios-crash-dump-analysis-book/zh/)


## class-dump

#### 构建过程
---

通常，当我们开发应用程序时，我们会将应用程序的 Debug 版本构建到我们的设备上。而当我们为测试人员、应用程序审核或应用商店构建应用时，我们构建应用程序的 Release 版本。

默认情况下，对于 Release版本，.o 文件的调试信息被存储在一个单独的目录结构中。被称作 our_app_name.DSYM。

当开发人员发现崩溃时，可以使用这些调试信息来帮助我们理解程序在哪里出错了。

当用户发现我们的应用程序崩溃时，并没有开发人员在身边。所以，会生成一份崩溃报告。它包含出现问题的机器地址。符号化可以将这些地址转换为有意义的源代码来作为参考。

为了进行符号化，必须拥有对应的 DSYM 文件。

默认情况下，Xcode 被设置为只为 Release 版本生成 DSYM 文件，Debug 版本则不会生成该文件。

#### 构建设置
---

| Setting | Meaning | Usually set for target |
| :---- | :----: | ----: |
| DWARF | 调试信息仅在 `.o` 文件中 | Debug |
| DWARF with dSYM | 除了 `.o`文件？也会将调试信息整理到DSYM文件中 | Release | 

#### DSYM 结构
---

DSYM 文件严格来说是一个目录层次结构：

>  icdab_planets.app.dSYM
icdab_planets.app.dSYM/Contents
icdab_planets.app.dSYM/Contents/Resources
icdab_planets.app.dSYM/Contents/Resources/DWARF
icdab_planets.app.dSYM/Contents/Resources/DWARF/icdab_planets
icdab_planets.app.dSYM/Contents/Info.plist

只是将通常放在 .o 文件中的 DWARF 数据，复制到另一个单独的文件中。

通过查看构建日志，我们可以看到 DSYM 是如何生成的。它实际上只是因为这个命令： 
`dsymutil path_to_app_binary -o output_symbols_dir.dSYM`

崩溃报告仅仅是更大的系统诊断报告的一部分。

#### Jetsam报告

Jetsam 一词最初是一个航海术语，指船只将不想要的东西扔进海里，以减轻船的重量。在 iOS 中，Jetsam 是将当前应用从内存中弹出以满足当前最重要应用需求的系统。

(“Identifying High-Memory Use with Jetsam Event Reports” 2020)

在 Jetsam 报告中首先要查找的是 reason 字段。

| Jetsam Reason |  Meaning  |
| :--- | --: |
|  per-process-limit | 已达到常驻内存限制。 该限制因应用程序或扩展程序的类型而异。  |
|  vm-pageshortage | 内核希望提供干净的页面给另一个进程，但是已经用完了，因此杀死了我们的进程。  |
|  vnode-limit | 内核已经用完了 vnode（UNIX文件的一种泛化），因此正在终止我们释放更多vnode的进程。 |
|  highwater | 进程使用过多的物理内存。 |
|  fc-thrashing | 对内存映射文件的过多随机访问导致了文件缓存的碎片/抖动。|
|  jettisoned | Jetsam 的其他原因。 |

#### 崩溃报告

##### iOS 崩溃报告 Header 部分

| Entry | Meaning |
| :--- | ---: |
|  Incident Identifier |  崩溃报告的唯一编号 |
| CrashReporter Key  |  崩溃设备的唯一标识符 |
| Hardware Model  |  Apple 硬件模型（iPad，iPhone） |
| Process  |  崩溃的进程名称（或编号） |
|  Path |  设备文件系统上崩溃程序的完整路径名 |
|  Identifier |  来自Info.plist 的 Bundle identifier |
| Version  |  CFBundleVersion；括号中有 CFBundleVersionString |
|  Code Type |  崩溃进程的目标体系结构 |
|  *Role* |  进程 task_role。如果我们在后台、前台或控制台应用程序中，都会显示一个指示器。主要影响进程的调度优先级。 |
|  Parent Process | 崩溃进程的父级。launchd 是一个进程启动程序，通常是父进程。  |
|  Coalition | 任务分组合并，然后他们就可以可以把资源消耗集中起来。  |

##### iOS 崩溃报告异常部分

> Exception Type:  EXC_CRASH (SIGKILL)
Exception Codes: 0x0000000000000000, 0x0000000000000000
Exception Note:  EXC_CORPSE_NOTIFY
Termination Reason: Namespace <0xF>, Code 0xdead10cc
Triggered by Thread:  0

| Entry | Meaning |
| :--- | ---: |
| Exception Type | Mach OS中的异常类型 |
| Exception Codes | 异常类型的编码，例如尝试访问无效的地址以及支持信息。 |
| Exception Note | 如果进程被看门狗计时器杀死，会显示SIMULATED（这不是崩溃），进程崩溃则显示 EXC_CORPSE_NOTIFY |
| Termination Reason | 视情况而定，它给出一个命名空间（数字或子系统名称）和一个 magic number（通常是一个看起来像英语单词的十六进制数字）。 有关每个终止代码的详细信息，请参见下文。 |
| Triggered by Thread | 导致崩溃的进程中的线程 |

###### 最重要的项是异常类型

| Exception Type  | Meaning  |
| :--- | ---: |
|  EXC_CRASH (SIGABRT) | 我们的程序触发了一个编程语言异常，例如失败的断言，这导致操作系统中止我们的应用程序 |
|  EXC_CRASH (SIGQUIT) | 一个进程从另一个正在管理它的进程接收到退出信号。通常，这意味着某个拓展程序花费了太长的时间或者消耗了太多的内存。App 的拓展程序仅能获得有限的内存。 |
| EXC_CRASH (SIGKILL) | 系统终止了我们的 App（或 App 的拓展程序），通常是因为已经达到了某种资源的限制。我们需要研究终止原因，以确定违反的某个政策是终止原因。 |
| EXC_BAD_ACCESS (SIGSEGV) 或 EXC_BAD_ACCESS (SIGBUS) | 我们的程序很可能试图访问错误的或者是我们没有权限访问的内存地址。或由于内存压力，该内存已被释放 |
| EXC_BREAKPOINT (SIGTRAP) | 这可能是由于触发了一个NSException（可能是我们自己的库触发的）或者是调用了_NSLockError 或 objc_exception_throw方法。例如，这可能是因为 Swift 检测到异常，例如强制展开 nil 可选 |
|  EXC_BAD_INSTRUCTION (SIGILL) | 这是程序代码本身有问题，而不是因为错误的内存访问。 这在 iOS 设备上应该很少见； 可能是编译器或优化器错误，或者是错误的手写汇编代码。 但在模拟器上，是不一样的，因为使用未定义的操作码是 Swift 运行时用来停止访问僵尸对象（已分配对象）的一种技术。 |
| EXC_GUARD | 这个问题发生在程序去关闭一个受保护的文件。例如，关闭系统使用的 SQLite 库 |

######  当存在终止原因时，我们可以按如下方式查找代码：

| Termination Code | Spoken Phrase | Meaning |
| :--- | :--: | ---: |
| 0xdead10cc | Deadlock| 我们在挂起之前持有文件锁或 sqlite 数据库锁。我们应该在挂起之前释放锁。 |
| 0xbaaaaaad | Bad| 通过侧面和两个音量按钮对整个系统进行了 stackshot。请参阅前面的系统诊断部分 |
| 0xbad22222 | Bad too many times| 可能是 VOIP 应用被频繁唤起导致的崩溃。也可以注意一下我们的后台调用网络的代码。 如果我们的TCP连接被唤醒太多次（例如 300 秒内唤醒 15 次），就会导致此崩溃。 |
| 0x8badf00d | Eat bad food | 我们的应用程序执行状态更改（启动、关闭、处理系统消息等）花费了太长时间。与Watchdog的时间策略发生冲突（超时）并导致终止。最常见的罪魁祸首是在主线程上进行同步的网络连接。 |
| 0xc00010ff | Cool Off| 系统检测的设备发烫而终止了我们的 App。如果只在少量设备上（几个）发生，那就可能是由于硬件的问题，而不是我们 App 问题。但是如果发生在其他设备上，我们应该使用 Instruments 去检查我们 App 的耗电量问题。 |
| 0x2bad45ec | Too bad for security | 发生安全冲突。 如果 Termination Description 显示为 Process detected doing insecure drawing while in secure mode，则意味着我们的应用尝试在不允许的情况下进行绘制，例如在锁定屏幕的情况下。 |

###### Aborts 

When we have a SIGABRT, we should look for what exceptions and assertions are present in our code from the stack trace of the crashed thread.

###### Memory Issues

当我们存在内存问题时， Exception Type 为 EXC_BAD_ACCESS 括号里是SIGSEGV 或者 SIGBUS，我们根据括号中的异常代码来判断是什么内存问题。对于这类问题，我们可以打开 Xcode 中关于特定 target scheme 的相关诊断程序。应该打开 address sanitizer，以查看是否可以发现错误。

> 选择 Edit Scheme 选择 Run 选择 Address Sanitizer

如果 Xcode 显示 App 正在使用大量内存，那么可能是我们所依赖的内存已被系统释放。为此，请打开 Malloc Stack 日志记录选项，选择 All Allocation And Free History。然后在 App 运行的某个时刻，可以单击 MemGraph 按钮，然后探索对象的分配历史记录。

###### Exceptions

当我们的异常类型为 EXC_BREAKPOINT 时，这可能会造成一定的困惑。该应用在没有 debugger 工具的情况下独自运行，那么断点从哪里来呢？通常，可能是因为我们运行了 NSException 代码。这使的系统在过程中跟踪陷阱信号，并使任何可用的调试器都这个过程中以帮助调试。所以即使我们在 debug 时断开断点，我们也会在这里停住，这样我们就可以找出存在运行时异常的原因。在正常的应用程序运行的情况下，没有 debugger 工具，系统只能崩溃应用程序。

###### 非法指令

当我们的异常类型为EXC_BAD_INSTRUCTION时，异常代码（接下来的字符）将是有问题的汇编代码。这种情况应改是罕见的。这值得我们去调整 Build Settings 中代码的优先级别，因为更高级别的优化可能会导致在构建期间发出更多奇特的指令，从而增加了编译器错误的机会。或者说，问题可能出在代码中具有手动编译优化功能的底层库中，例如多媒体资源库。手写汇编指令可能是错误出现的原因。

###### Guard exceptions

Certain files descriptors on the system are specially protected because they are used by the Operating System. When such file descriptors are closed (or otherwise modified) we can get a EXC_GUARD exception.

##### iOS 崩溃报告已过滤的 syslog 部分例外

> Filtered syslog:
None found

这是一个异常部分，因为他会去查看崩溃进程的进程 ID，然后查看该进程是否有任何 syslog （系统日志）部分。这个例子中我们并未在崩溃报告看到任何已过滤的信息，只看到 None found 。

##### iOS 崩溃报告线程状态部分

##### iOS 崩溃报告的 Binary Images 部分

## Xcode-diagnostics
## Xcode Static Analyzer 

## Pointer Authentication

使用 Apple A12 芯片或更高版本的设备将称为 Pointer Authentication 的安全功能作为 ARMv8.3-A 体系结构的一部分。例如， iPhone XS，iPhone XS Max, 和 iPhone XR 使用 A12芯片。其基本思想是在 64 位指针中存在未使用的位，因为它已经可以寻址相当广泛的地址，因此只有 40 位被分配给这样的目的。 (“Examining Pointer Authentication on the iPhone Xs” 2019) 因此，剩余的位可用于存储在将预期的指针地址与上下文值和密钥组合后计算出的哈希值。 然后，如果由于错误或恶意操作而要更改指针地址，则该指针将被认为是无效的，并且如果将其用于更改程序的控制流，则最终将导致 SIGSEGV。

## Classical Crash

### Runtime Environment Crashes
### Bad Memory Crashes

#### General Principles

---

In operating systems, memory is managed by first collating contiguous memory into memory pages, and then collating pages into segments. This allows metadata properties to be assigned to a segment that applies to all pages within the segment. This allows the code of our program (the program TEXT) to be set to read only but executable. This improves performance and security.

**SIGBUS** (bus error) means the memory address is correctly mapped into the address space of the **process**, but the process is not allowed to access the memory.

SIGSEGV (segment violation) means the memory address is not even mapped into the process address space.



### Application Abort Crashes

#### General principle

---

These crashes are distinguished by reporting Exception Type, **EXC_CRASH (SIGABRT)** in their Crash Report.

Many Operating System language support modules, and libraries, have code to detect fatal programming errors. In such circumstances, the application requests that the Operating System terminate the app. This gives rise to a SIGABRT crash.

### Resource Crashes
1. CPU Usage Crash
2. Wake Up Crash
3. Wake Up Crash Exception 
4. Temperature Crash

### Application Termination Crashes

#### General principle

---

These crashes are distinguished by reporting Exception Type, **EXC_CRASH (SIGKILL)** in their Crash Report.

1. Deadlock Crash
2. Insecure Drawing Crash

## Memory Diagnostics


```
// Define the type of the lag
typedef NS_ENUM(NSUInteger, EDumpType) {
    EDumpType_Unlag = 2000,
    EDumpType_MainThreadBlock = 2001,            // foreground main thread block
    EDumpType_BackgroundMainThreadBlock = 2002,  // background main thread block
    EDumpType_CPUBlock = 2003,                   // CPU too high
    //EDumpType_FrameDropBlock = 2004,             // frame drop too much,no use currently
    //EDumpType_SelfDefinedDump = 2005,             // no use currently
    //EDumpType_B2FBlock = 2006,                   // no use currently
    EDumpType_LaunchBlock = 2007,                // main thread block during the launch of the app
    //EDumpType_CPUIntervalHigh = 2008,            // CPU too high within a time period
    EDumpType_BlockThreadTooMuch = 2009,         // main thread block and the thread is too much. (more than 64 threads)
    EDumpType_BlockAndBeKilled = 2010,           // main thread block and killed by the system
    //EDumpType_JSStack = 2011,                    // no use currently
    EDumpType_PowerConsume = 2011,                // battery cost stack report
    EDumpType_Test = 10000,
};
```

专业市场订单密度高，是货运订单的重要来源。也是线下BD拓展和维护商户的重要“战场”。
如何通过技术能力助力业务“打透”专业市场，优化专业市场供需效率。是我们一直在思考的问题。
经过【靶心】- 市场地图的探索、线下市场的走访和行业信息的调研，
逐步形成对专业市场建模的思路。
