## Kraken 是什么

Kraken 是一款基于 W3C 标准的高性能渲染引擎。Kraken 底层基于 Flutter 进行渲染，通过其自绘渲染的特性，保证多端一致性。上层基于 W3C 标准实现，拥有非常庞大的前端开发者生态。


## 与浏览器的差异

### 使用JS而不是HTML

Web 的入口文件是一个 **.html** 或 **.htm** 等为扩展名的 HTML 文档。

而 Kraken 更类似 React Native 和 Weex，它接受一个 **.js** 的 JSBundle，使用 DOM API 构建视图和样式。

这样的选择是出于性能的考虑，现代的前端框架通常使用 JS 逻辑操作 DOM 生成 UI，直接下载 JS 可以减少下载解析 HTML 文档所花费的时间。

### 有限的CSS支持

### 有限的 DOM & 全局 API 支持

### 样式能力差异

### 本地存储