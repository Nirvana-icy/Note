[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)

## Runloop的概念
### CFRunloopRef 是在CoreFoundation框架内，提供了纯C函数的API，所有这些API都是线程安全的。
### NSRunloop 基于CFRunloopRef封装，提供了面向对象的API，但这些API不是线程安全的。

## Runloop与线程的关系

### 获取主线程
#### pthread_main_thread_np()
#### [NSThread mainThread]

### 获取当前线程
#### pthread_self()
#### [NSThread currentThread] 

### 苹果不允许直接创建 RunLoop，它只提供了两个自动获取的函数
#### CFRunLoopGetMain()
#### CFRunLoopGetCurrent()

### Runloop对外的接口

#### CFRunLoopRef

#### CFRunLoopModeRef

#### CFRunLoopSourceRef

##### Source0

只包含了一个回调（函数指针），它并不能主动触发事件。

##### Source1

包含一个mach_port和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。

这种Source可以主动唤醒RunLoop的线程。

#### CFRunLoopTimerRef

是基于时间的触发器，它和NSTimer是toll-free bridged的，可以混用。其包含一个时间长度和一个回调（函数指针）。

当其加入到RunLoop时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

#### CFRunLoopObserverRef

是观察者，每个Observer都包含了一个回调（函数指针）。

当RunLoop的状态发生变化时，观察者就能通过回调接受到这个变化。

### RunLoop的Mode

### RunLoop的内部逻辑

实际上RunLoop就是这样一个函数，其内部是一个do-while循环。

当你调用CFRunLoopRun()时，线程就会一直停留在这个循环里；

知道超时或被手动停止，该函数才会返回。

### RunLoop的底层实现

RunLoop的核心是基于mach port的，其进入休眠时调用的函数是mach_msg()。

和其他架构不同，Mach的对象间不能直接调用，只能通过消息传递的方式实现对象间的通信。

“消息”是Mach中最基础的概念，消息在两个端口之间传递，这就是Mach IPC的核心。

RunLoop的核心就是一个 mach_msg()，RunLoop调用这个函数去接收消息。

如果没有别人发送port消息过来，内核会将线程置于等待状态。

例如你在模拟器里跑起一个iOS的App，然后在App静止时点击暂停，

你会看到主线程调用栈是停留在mach_msg_trap()这个地方。

### 苹果用RunnLoop实现的功能

#### AutoreleasePool

#### 事件响应

硬件事件（触摸/锁屏/摇晃）首先由IOKit.framework -生成-> IOHIDEvent -> SpringBoard接收 -> mach port转发给需要的App进程

触发Source1注册的回调 -> _IOHIDEventSystemClientQueueCallback() -调用-> _UIApplicationHandleEventQueue() 进行应用内部的分发。

_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，

其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。

通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

#### 手势识别

#### 界面更新

#### 定时器

#### PerformSelecter

#### GCD

#### 网络请求







