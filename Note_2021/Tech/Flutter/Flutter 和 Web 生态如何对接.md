## [Flutter 和 Web 生态如何对接？](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247494044&idx=1&sn=33372c3858c64a1ea9c4050453959ce1&chksm=e92ad493de5d5d8559ef688ce2b51962e00ef187c4f62c15715bd6f48804d148657d83dcf81c&cur_album_id=1404584922274398210&scene=189#rd)

### 为什么要对接

#### 前端开发者使用 Flutter 的各方面成本：

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBFibjQYrNiaq80pRVZsNvvsaPHXxpzIqZ9ufVibRJrZUiaqZ51F6gBsJibl6g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 关于 Flutter 和 Web 生态的对接涉及两个方面：

1. 从 Web 到 Flutter。就是使用 Web 技术栈来开发，然后对接到 Flutter 上实现跨平台渲染。对 Web 来说是解决性能和跨平台一致性问题，对 Flutter 来说是解决生态复用问题。

2. 从 Flutter 到 Web。就是官方已经实现的 Web support for Flutter，把已经用 Dart 开发好的 App 编译成 HTML/JS/CSS 然后运行在浏览器上，可以用于降级和外投场景。


### 如何实现“从 Web 到 Flutter”？

#### 首先分析一下 Flutter 的架构图，看看可以从哪里下手。

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBFuNrwTxq7hGDaLn3QG3iaiasSWtmbXDUoibBk0oeGn6OEbZ6bebzbSXUjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Flutter 可以分为 Framework 和 Engine 两部分，Engine 部分比较底层也比较稳定了，最好不要动，需要改的是用 Dart 实现的 Framework。**

要想对接 Web 生态的话，**JS 引擎肯定是要引入的**，至于是否保留 Dart VM 有待讨论。

图中最上面 Material 和 Cupertino 两个 UI 库前端是不需要的，前端有自己的。

**关键是 Widget 这部分**，

1. 是替换成 HTML/CSS 的方式写 UI? 

2. 还是继续保留 Widget 但是把语言换成 JS? 

不同方案给出的解法也不一样。

#### 有不少方案可以实现对接，业界有挺多尝试的，作者总结了下面三种方式：

1. TS 魔改：用 JS 引擎替换掉 Dart VM，用 JS/TS 重新实现 Flutter Framework（或者直接 dart2js 编译过来）。

2. JS 对接：引入 JS 引擎同时保留 Dart VM，用前端框架对接 Flutter Framework。

3. C++ 魔改：用 JS 引擎替换掉 Dart VM，用 C++ 重新实现 Flutter Framework。

#### 几种方案对比

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBFLvIlGbGcaa4PibmYMRYiaibhxahOsNic47cLDSFosdrvz7mciauTB0779Vw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图中实线部分表示了跨语言的通信，太过频繁会影响性能，虚线部分表示了其他对接可能性。

从下到上，Flutter Engine 是不需要动的，这一层是跨平台的关键。

Framework 则有三种语言版本，JS/TS、Dart、C++，性能是 C++ 版本最好，成本是 Dart 版本最低。

然后还需要向上处理 HTML/CSS 和 Widget 的问题，

可以直接对接一个前端框架，也可以直接在 C++ 层实现（不然需要透出的 binding 接口就太多了，用通信的方式也太过频繁了）。

### 如何实现“从 Flutter 到 Web”？

这个功能官方已经实现了，可以把使用 Dart 开发的 App 编译成 Web App 运行在浏览器上，官方文档以介绍用法和 API 为主，我这里简单分析一下内部具体的实现方案。

#### 实现原理

结合 Flutter 的架构图来看，要实现 Web 到 Flutter 需要改造的是上层 Framework，

要实现 Flutter 到 Web 需要改造的则是底层 Engine。

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBFtcoLEwfgNoc1TrcASCdjCuTZ2stkC8m6WpCy7uk6u1hLH2EnLyZwOw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Framework 对 Engine 的核心依赖是 dart:ui，dart:ui库是在 Engine 里实现的，

抽象出了绘制 UI 图层的接口，底层对接 skia 的实现，向上透出 Dart 语言的接口。

这样来看，对接方式就比较简单了：

1. 使用 dart2js 把 Framework 编译成 JS 代码。

2. 基于浏览器的 API 重新实现 dart:ui，即 dart:web_ui。

把 Dart 编译成 JS 没什么问题，性能可能会有一点影响，功能都是可以完全保留的.

关键是 dart:web_ui 的实现。

在原生 Engine 中，dart:ui 依赖 skia 透出的 SkCanvas 实现绘制，这是一套很底层的图形接口，只定义了画线、画多边形、贴图之类的底层能力.

用浏览器接口实现这一套接口还是很有挑战的。

上图可以看到 Web 版 Engine 是基于 DOM 和 Canvas 实现的，

底层定义了 DomCanvas 和 BitmapCanvas 两种图形接口，会把传来的 layer tree 渲染成浏览器的 Element tree，

但是节点上仅包含了 position, transform, opacity 之类的样式，只用到 CSS 很小的一个子集，一些更复杂的绘制直接用 2D <canvas\> 实现。

### 一种适应性更好的架构

如果理想化一点，能不能从架构角度让 Flutter 和 Web 生态融合的更好一些呢？

回顾文章最开始的官方架构图，上面是 Framework(Dart)，下面是 Engine(C++)。

切分在 Foundation 这一层，双方之间的交互是几何图形信息。

如果还保持这个架构，把切分层次划分的更靠上一些，如下图所示，划分在 Widgets 和 Rendering 这一层。

理论上讲对 Flutter 的开发者来说是无感知的，因为上层的开发语言和 Widget 接口都是不变的。

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBF3wzQqY4X4FofNL5JTEF3E9CsCSvRVdc3P1xjWVvr5AgXefRbEExexA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

切分在这一层，Framework 和 Engine 之间的交互就不再是几何图形而是节点信息。

Widget 的组合、setState 响应式更新、Widget diff 都还在 Dart 中。

展开后的 RenderObject 的布局、绘制、裁剪、动画全都在 C++ 中，不仅有更好的性能，还可以与 Engine 有更好的结合。


或者说，还原本保留 Engine 的设计，**把下沉的这部分逻辑上划分成 Render**，就有了如下三层的结构：

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBFoYbqhkW0nznOvCRFdefwRicsBOngk38ymmMs6glPZicPatet0uSzmGibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样划分出来的每一层都有明确的定位：

1. Framework: 开发框架。为开发者提供可编程 API，实现响应式的开发模式，提供细粒度 Widget 供开发者自由封装和组合。

2. Renderer: 渲染引擎。专门实现布局、绘制、动画、手势的的处理，这部分功能相对独立，是可以与开发框架解耦的，也不必与特定语言绑定。

3. Engine: 图形引擎。实现跨平台一致的图形接口，合成输入的层并绘制到屏幕上，处理好平台力的接入和适配。

---

这样切分除了有性能优势以外，也使得渲染引擎摆脱了对 Dart 的依赖。能够支持多种语言，也能支持多种开发模式。

对接到 Dart VM 就可以用 Dart 写代码，

对接到 JS 引擎就可以用 JS 写代码，

对接到 JVM 还可以写 Java，

---

但是无论怎么写，底层的渲染能力是一样的，一套统一的布局算法，动画和手势的处理行为也是一致的。

在这样的架构下，对接 Web 生态就更容易了。

Dart 和 Widget 是前端不想要的，希望能换成 JS 和 CSS，但是又想要底层的跨平台一致渲染引擎，

那从 Renderer 层开始对接就好了，绕过了所有不想要的，也保留了所有想要的。

![image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ6V2Fksbd6bKfEcWZzPDBF2QsxaG2zibPKyaOmMicdiaDQYF66sFYTDpWZavXfDbtSzy7t7rP9B8C4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

要实现 Flutter for Web 也更简单了一些。

在 Engine 层做对接，一直苦于浏览器透出的底层能力不够。

如果是在 Renderer 之上做对接就更容易一些，

基于 JS/CSS/DOM/Canvas 的能力封装出一套 Rendering 接口，供 Widget 调用就好了。

这样可以使渲染链路更短一些，但是依然要处理 Widget 和 DOM/CSS 之间的兼容性问题。