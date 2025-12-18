# 第 1 章：Sizer 概述与设计思想

## 引言

在开始深入源码之前，我们需要先理解 Sizer 要解决什么问题，以及它的设计哲学。本章将为你建立全局视角，为后续深入源码打下坚实基础。

## 学习目标

- 理解 Sizer 解决的核心问题
- 掌握 Sizer 的设计目标和原则
- 了解 Sizer 的整体架构
- 对比 Sizer 与其他响应式方案

## Sizer 解决的问题

### 移动端适配的挑战

在 Flutter 开发中，我们面临的主要挑战是：

1. **屏幕尺寸多样性**：从 4 英寸的 iPhone SE 到 12.9 英寸的 iPad Pro
2. **像素密度差异**：不同设备的 DPI（每英寸像素数）差异巨大
3. **横竖屏切换**：同一应用需要在不同方向下都能良好显示
4. **平台差异**：Android、iOS、Web、桌面端的显示特性不同

传统的硬编码像素值（如 `width: 100`）在这些场景下会导致：

- 小屏设备上 UI 元素过大
- 大屏设备上 UI 元素过小
- 横竖屏切换时布局错乱
- 不同设备上视觉比例不一致

### Sizer 的解决方案

Sizer 通过以下方式解决这些问题：

1. **百分比布局**：使用屏幕尺寸的百分比而非固定像素
2. **扩展方法语法**：提供直观的 `.h`、`.w`、`.sp`、`.dp` 等扩展方法
3. **自动适配**：根据设备特性自动计算合适的尺寸
4. **多平台支持**：统一 API 支持移动端、Web 和桌面端

## 设计目标

### 简洁性

Sizer 追求极简的 API 设计，让开发者能够：

```dart
Container(
  width: 50.w,  // 屏幕宽度的 50%
  height: 30.h, // 屏幕高度的 30%
)
```

对比传统的 MediaQuery 方式：

```dart
Container(
  width: MediaQuery.of(context).size.width * 0.5,
  height: MediaQuery.of(context).size.height * 0.3,
)
```

### 一致性

Sizer 确保不同设备上相同的代码产生一致的视觉比例：

- 使用百分比而非像素值
- 统一的计算基准（屏幕尺寸）
- 自动处理安全区域（SafeArea）

### 性能

Sizer 采用静态变量存储设备信息，避免频繁的 MediaQuery 查询：

- 初始化时计算一次设备信息
- 后续使用直接读取静态变量
- 减少 Widget 重建时的计算开销

## 整体架构

Sizer 的核心架构包含四个主要组件：

### 1. Sizer Widget

`Sizer` 是应用的入口 Widget，负责：

- 初始化设备信息
- 监听布局变化
- 监听方向变化
- 触发响应式更新

### 2. Device 类

`Device` 类存储和管理设备信息：

- 屏幕尺寸（width、height）
- 安全区域尺寸（safeWidth、safeHeight）
- 设备类型（android、ios、web 等）
- 屏幕类型（mobile、tablet、desktop）
- 方向（portrait、landscape）
- 像素密度和宽高比

### 3. 扩展方法

`SizerExt` 扩展为 `num` 类型提供响应式计算方法：

- `.h`、`.w`：基于屏幕尺寸的百分比
- `.sh`、`.sw`：基于安全区域的百分比
- `.sp`、`.dp`：基于像素密度的缩放
- `.vmin`、`.vmax`：基于视口尺寸
- 绝对单位转换（cm、mm、inches 等）

### 4. Adaptive 类

`Adaptive` 提供静态方法版本的响应式计算，作为扩展方法的替代方案。

## 与 Flutter 原生方案的对比

### MediaQuery

Flutter 原生的 `MediaQuery` 需要：

```dart
MediaQuery.of(context).size.width * 0.5
```

**优点**：

- 官方支持，稳定可靠
- 功能全面
- 与 Flutter 框架深度集成

**缺点**：

- API 冗长
- 需要 BuildContext
- 每次使用都需要查询

### LayoutBuilder

`LayoutBuilder` 提供布局约束信息：

```dart
LayoutBuilder(
  builder: (context, constraints) {
    return Container(width: constraints.maxWidth * 0.5);
  },
)
```

**优点**：

- 精确的约束信息
- 不需要 MediaQuery
- 性能较好

**缺点**：

- 代码嵌套深
- 只提供约束，不提供设备信息

### Sizer 的优势

1. **语法简洁**：`50.w` 比 `MediaQuery.of(context).size.width * 0.5` 简洁得多
2. **全局可用**：扩展方法在任何地方都可以使用
3. **功能丰富**：提供多种计算方式（百分比、像素密度、绝对单位等）
4. **性能优化**：静态变量缓存，避免重复计算

## 核心设计模式

### 单例模式（静态类）

`Device` 类使用静态变量存储全局设备信息：

```dart
class Device {
  static late double height;
  static late double width;
  // ...
}
```

这种设计使得：

- 设备信息全局可访问
- 避免重复计算
- 内存占用小

### 扩展方法模式

通过扩展方法增强 `num` 类型的功能：

```dart
extension SizerExt on num {
  double get h => this * Device.height / 100;
  double get w => this * Device.width / 100;
}
```

这种设计使得：

- API 直观易用
- 类型安全
- 支持链式调用

### 构建器模式

`Sizer` Widget 使用构建器模式：

```dart
Sizer(
  builder: (context, orientation, screenType) {
    return MaterialApp(/* ... */);
  },
)
```

这种设计使得：

- 灵活的构建逻辑
- 响应式更新机制
- 清晰的依赖关系

## 使用场景

Sizer 适用于以下场景：

1. **移动应用**：需要适配多种屏幕尺寸的移动应用
2. **响应式 Web**：需要在不同浏览器窗口尺寸下良好显示的 Web 应用
3. **桌面应用**：需要适配不同显示器尺寸的桌面应用
4. **多平台应用**：同一代码库需要支持多个平台的 Flutter 应用

## 总结

Sizer 通过简洁的 API 和高效的实现，解决了 Flutter 开发中的响应式适配问题。其核心设计思想包括：

- **简洁优先**：API 设计追求简洁直观
- **性能考虑**：使用静态变量缓存减少计算
- **功能丰富**：提供多种计算方式满足不同需求
- **平台统一**：统一的 API 支持多平台

## 检查清单

在进入下一章之前，确保你理解：

- [ ] Sizer 要解决的核心问题是什么
- [ ] Sizer 的设计目标和原则
- [ ] Sizer 的整体架构包含哪些组件
- [ ] Sizer 相比 Flutter 原生方案的优势
- [ ] Sizer 使用了哪些设计模式

## 实践练习

1. 思考一下，如果你不使用 Sizer，如何实现类似的响应式布局？会有什么困难？

2. 对比 Sizer 的 `.h`、`.w` 方法与 MediaQuery 的使用，分析各自的优缺点。

3. 设想一个场景，Sizer 可能无法很好地满足需求，你会如何扩展它？
