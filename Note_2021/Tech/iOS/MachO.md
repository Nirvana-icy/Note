### 什么是符号

![image](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e534b72ff3914e7a870f0a4a31fa781e~tplv-k3u1fbpfcp-watermark.image)

1. 符号表
2. 间接符号表
3. 字符串表

### 符号绑定

#### dyld的加载流程（应用程序的启动）

![iamge](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2701eda5aefb46628bcfd8e3157871a5~tplv-k3u1fbpfcp-watermark.image)

[iOS底层探索--dyld与objc的关联](https://juejin.cn/post/6883077259171725319)

### [fishhook原理](https://zhuanlan.zhihu.com/p/342640035)

MachO 文件内部存有代码和数据，代码放在 _TEXT 段（代码段）中，数据放在 _DATA 段（数据段）中。

数据段除了全局变量、常量、自定义类还有很多东西，比如我们刚才提到的符号表。

