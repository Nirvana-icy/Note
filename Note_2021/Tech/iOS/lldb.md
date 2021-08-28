#### expr

1. po -- 只打印数值
2. p -- 还打印变量的类型

3. p/<fmt> -- 指定打印格式   eg. p/x number

---

#### image lookup

image lookup -a
image lookup -n
image lookup -t
image list

---

#### thread  查看线程堆栈信息

1. thread backtrace & bt 查看堆栈调用

2. up down 查看上一步的堆栈调用信息

3. frame select 编号，跳转至指定堆栈查看，源码和汇编（系统的或打包的），定位某个方法的具体实现。

4. frame variable  查看方法所有参数

#### watchpoint 内存断点

`watchpoint set variable self->age`
`watchpoint delete`


---

#### memory read

`memory read 0x10074c450`
`// 简写  x 0x10074c450`
 
// 增加读取条件
// memory read/数量 格式 字节数  内存地址
// 简写 x/数量 格式 字节数  内存地址
// 格式 x是16进制，f是浮点，d是10进制
// 字节大小   b：byte 1字节，h：half word 2字节，w：word 4字节，g：giant word 8字节
 
`示例：x/4xw    //   /后面表示如何读取数据 w表示4个字节4个字节读取，x表示以16进制的方式读取数据，4则表示读取4次`

#### register read

register read x0


NSObject *obj = [[NSObject alloc] init];

objc_storeStrong

####

weak 所引用对象的计数器不会加1，并在引用对象被释放的时候自动被设置为 nil。

####
(lldb) image lookup -t ViewController    // -t 查看某个符号的定义   -s 查看某个符号的位置



