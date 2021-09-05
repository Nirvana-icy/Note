### 卡顿监控

在 iOS/macOS 平台应用中，主线程有一个 Runloop。

Runloop 是一个 Event Loop 模型，让线程可以处于接收消息、处理事件、进入等待而不马上退出。

在进入事件的前后，Runloop 会向注册的 Observer 通知相应的事件。

Matrix 卡顿监控在 Runloop 的起始最开始和结束最末尾位置添加 Observer，从而获得主线程的开始和结束状态。

1. 卡顿监控起一个子线程定时检查主线程的状态，当主线程的状态运行超过一定阈值则认为主线程卡顿，从而标记为一个卡顿。

2. 单CPU  80% 

Dump 堆栈到内存

Dump 内存到文件

#### 耗时堆栈提取



Metrixkit

```
// delay one second
    s_foregroundDelayBlock = dispatch_block_create(DISPATCH_BLOCK_NO_QOS_CLASS, ^{
       
    });
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), s_foregroundDelayBlock);
```


[Matrix Inside](https://www.randomj.top/2019/05/28/matrix-share/)

[异步堆栈回溯](https://github.com/Tencent/matrix/wiki/Matrix-for-iOS-macOS-异步堆栈回溯)

1. MatrixAsyncHook > beginHook - hook dispatch > record trace in static dict 

2. dispatch task with block or func to queue > GCD dispatch the tast to some thread > record trace in static dict > repeat... > report dict

3. 

```
static inline dispatch_block_t blockRecordAsyncTrace(dispatch_block_t block)
{
    // 1. get origin stack
    AsyncStackTrace stackTrace = getCurAsyncStackTrace();
    NSMutableArray *stackArray = [[NSMutableArray alloc] init];
    for (int i = 0; i < stackTrace.size; i++) {
        NSNumber *temp =[NSNumber numberWithUnsignedLong:(unsigned long)stackTrace.backTrace[i]];
        [stackArray addObject:temp];
    }
    free(stackTrace.backTrace);
    stackTrace.backTrace = NULL;
    
    // 2. execute the block
    dispatch_block_t newBlock = ^() {
        pthread_mutex_lock(&m_threadLock);
        thread_t current_thread = (thread_t)ksthread_self();
        NSNumber *key = [[NSNumber alloc] initWithInt:current_thread];
        [asyncOriginThreadDict setObject:stackArray forKey:key];
        pthread_mutex_unlock(&m_threadLock);
        
        block();
        
        pthread_mutex_lock(&m_threadLock);
        if (key != nil && [asyncOriginThreadDict objectForKey:key] != nil) {
            [asyncOriginThreadDict removeObjectForKey:key];
        }
        pthread_mutex_unlock(&m_threadLock);
    };
    
    // 3. return the new block
    return newBlock;
}
```