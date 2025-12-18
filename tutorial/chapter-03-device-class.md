# 第 3 章：Device 类的实现原理

## 引言

`Device` 类是 Sizer 的核心，它负责收集、存储和管理设备信息。理解 `Device` 类的实现，是掌握 Sizer 工作原理的关键。本章将深入分析 `Device` 类的每个细节。

## 学习目标

- 理解 `Device` 类的静态变量设计
- 掌握 `setScreenSize` 方法的完整流程
- 理解 SafeArea 的计算逻辑
- 掌握设备类型和屏幕类型的判断算法
- 理解 BoxConstraints 和 MediaQuery 的使用

## Device 类概览

让我们先看看 `Device` 类的完整结构：

```dart 13:113:lib/util.dart
class Device {
  /// Device's BoxConstraints
  static late BoxConstraints boxConstraints;

  /// Device's Orientation
  static late Orientation orientation;

  /// Type of Device
  static late DeviceType deviceType;

  /// Type of Screen
  static late ScreenType screenType;

  /// Device's Height
  static late double height;

  /// Device's Width
  static late double width;

  // TODO: Reconsider if we should include AppBar and BottomNavigationBar
  // in safe height calculation

  /// Device's Remaining Height after applying `SafeArea`
  static late double safeHeight;

  /// Device's Remaining Width after applying `SafeArea`
  static late double safeWidth;

  /// Device's Aspect Ratio
  static late double aspectRatio;

  /// Device's Pixel Ratio
  static late double pixelRatio;

  /// Sets the Screen's size and Device's `Orientation`,
  /// `BoxConstraints`, `Height`, and `Width`
  static void setScreenSize(
    BuildContext context,
    BoxConstraints constraints,
    Orientation currentOrientation,
    double maxMobileWidth, [
    double? maxTabletWidth,
  ]) {
    // Sets boxconstraints and orientation
    boxConstraints = constraints;
    orientation = currentOrientation;

    // Sets screen width and height
    width = boxConstraints.maxWidth;
    height = boxConstraints.maxHeight;

    // Calculates remaining available size after `SafeArea`
    // final viewPadding = MediaQuery.of(context).viewPadding;
    final viewPadding = MediaQuery.viewPaddingOf(context);
    safeWidth = width - (viewPadding.left + viewPadding.right);
    safeHeight = height - (viewPadding.top + viewPadding.bottom);

    // Sets aspect and pixel ratio
    aspectRatio = constraints.constrainDimensions(width, height).aspectRatio;
    // pixelRatio = MediaQuery.of(context).devicePixelRatio;
    pixelRatio = MediaQuery.devicePixelRatioOf(context);

    // Sets DeviceType
    if (kIsWeb) {
      deviceType = DeviceType.web;
    } else {
      switch (defaultTargetPlatform) {
        case TargetPlatform.android:
          deviceType = DeviceType.android;
          break;
        case TargetPlatform.iOS:
          deviceType = DeviceType.ios;
          break;
        case TargetPlatform.windows:
          deviceType = DeviceType.windows;
          break;
        case TargetPlatform.macOS:
          deviceType = DeviceType.mac;
          break;
        case TargetPlatform.linux:
          deviceType = DeviceType.linux;
          break;
        case TargetPlatform.fuchsia:
          deviceType = DeviceType.fuchsia;
          break;
      }
    }

    // Sets ScreenType
    if ((orientation == Orientation.portrait && width <= maxMobileWidth) ||
        (orientation == Orientation.landscape && height <= maxMobileWidth)) {
      screenType = ScreenType.mobile;
    } else if (maxTabletWidth == null ||
        (orientation == Orientation.portrait && width <= maxTabletWidth) ||
        (orientation == Orientation.landscape && height <= maxTabletWidth)) {
      screenType = ScreenType.tablet;
    } else {
      screenType = ScreenType.desktop;
    }
  }
}
```

## 枚举类型定义

在分析 `Device` 类之前，我们先看看相关的枚举类型：

```dart 3:11:lib/util.dart
/// Type of Device
///
/// This can be android, ios, fuchsia, web, or desktop (windows, mac, linux)
enum DeviceType { android, ios, fuchsia, web, windows, mac, linux }

/// Type of Screen
///
/// This can either be mobile or tablet
enum ScreenType { mobile, tablet, desktop }
```

这些枚举定义了设备类型和屏幕类型的可能值。

## 静态变量设计

### 为什么使用静态变量？

`Device` 类的所有成员都是 `static`，这种设计有以下原因：

1. **全局单例**：设备信息在整个应用中只有一份，不需要多个实例
2. **全局访问**：可以在任何地方直接访问，如 `Device.height`、`Device.width`
3. **内存效率**：只占用一份内存空间
4. **初始化控制**：通过 `setScreenSize` 统一初始化

### 使用 late 关键字

所有静态变量都使用 `late` 关键字：

```dart
static late double height;
static late double width;
```

**`late` 的作用**：

- 允许非空变量延迟初始化
- 在 `setScreenSize` 方法中初始化
- 避免了使用可空类型（`double?`）的复杂性

**为什么不使用 `double?`**：

如果使用 `double?`，每次访问都需要空检查：

```dart
static double? height;

// 使用时需要
Device.height ?? 0.0  // 每次都要检查
```

使用 `late` 后，一旦初始化完成，就可以安全使用：

```dart
Device.height  // 直接使用，无需检查
```

**潜在风险**：

如果在 `setScreenSize` 之前访问这些变量，会抛出 `LateInitializationError`。但这是预期的行为，因为设备信息应该在应用启动时初始化。

## 核心方法：setScreenSize

`setScreenSize` 是 `Device` 类的核心方法，负责初始化所有设备信息。让我们逐步分析：

### 方法签名

```dart 49:55:lib/util.dart
  static void setScreenSize(
    BuildContext context,
    BoxConstraints constraints,
    Orientation currentOrientation,
    double maxMobileWidth, [
    double? maxTabletWidth,
  ]) {
```

**参数说明**：

- `context`：BuildContext，用于访问 MediaQuery
- `constraints`：BoxConstraints，提供布局约束信息
- `currentOrientation`：当前屏幕方向
- `maxMobileWidth`：移动设备的最大宽度阈值（默认 599）
- `maxTabletWidth`：平板设备的最大宽度阈值（可选，用于区分平板和桌面）

### 第一步：设置基础约束和方向

```dart 56:58:lib/util.dart
    // Sets boxconstraints and orientation
    boxConstraints = constraints;
    orientation = currentOrientation;
```

这里直接保存传入的约束和方向信息。

**BoxConstraints 的作用**：

`BoxConstraints` 提供了布局约束信息，包含：

- `minWidth`、`maxWidth`：宽度约束
- `minHeight`、`maxHeight`：高度约束

我们主要使用 `maxWidth` 和 `maxHeight` 作为屏幕尺寸。

### 第二步：设置屏幕尺寸

```dart 60:62:lib/util.dart
    // Sets screen width and height
    width = boxConstraints.maxWidth;
    height = boxConstraints.maxHeight;
```

这里直接使用约束的最大宽度和高度作为屏幕尺寸。

**为什么使用 maxWidth/maxHeight**：

- `maxWidth` 和 `maxHeight` 代表可用空间的边界
- 对于全屏应用，这就是屏幕的宽高
- 即使有嵌套的 LayoutBuilder，也能正确获取尺寸

### 第三步：计算安全区域

```dart 64:68:lib/util.dart
    // Calculates remaining available size after `SafeArea`
    // final viewPadding = MediaQuery.of(context).viewPadding;
    final viewPadding = MediaQuery.viewPaddingOf(context);
    safeWidth = width - (viewPadding.left + viewPadding.right);
    safeHeight = height - (viewPadding.top + viewPadding.bottom);
```

**SafeArea 的概念**：

SafeArea 是屏幕上的安全显示区域，排除了：

- 状态栏（Status Bar）
- 导航栏（Navigation Bar）
- 刘海屏的缺口
- 其他系统 UI 元素

**计算逻辑**：

```dart
safeWidth = width - (viewPadding.left + viewPadding.right)
safeHeight = height - (viewPadding.top + viewPadding.bottom)
```

- `viewPadding.left` 和 `viewPadding.right`：左右两侧被占用的宽度
- `viewPadding.top` 和 `viewPadding.bottom`：上下两侧被占用的高度
- 安全区域的尺寸 = 屏幕尺寸 - 被占用的尺寸

**代码注释中的 TODO**：

```dart 32:33:lib/util.dart
  // TODO: Reconsider if we should include AppBar and BottomNavigationBar
  // in safe height calculation
```

这个 TODO 提出了一个设计问题：是否应该在安全高度计算中考虑 AppBar 和 BottomNavigationBar？

当前实现不考虑它们，因为：

- AppBar 和 BottomNavigationBar 是应用级别的 UI，不是系统 UI
- 不同的应用可能有不同的布局结构
- 保持计算的通用性

### 第四步：设置宽高比和像素密度

```dart 70:73:lib/util.dart
    // Sets aspect and pixel ratio
    aspectRatio = constraints.constrainDimensions(width, height).aspectRatio;
    // pixelRatio = MediaQuery.of(context).devicePixelRatio;
    pixelRatio = MediaQuery.devicePixelRatioOf(context);
```

**宽高比（Aspect Ratio）**：

```dart
aspectRatio = constraints.constrainDimensions(width, height).aspectRatio
```

- `constrainDimensions` 方法会根据约束调整尺寸
- `aspectRatio` 是宽度与高度的比值
- 例如：1920x1080 的屏幕，宽高比是 16:9 = 1.778

**像素密度（Pixel Ratio）**：

```dart
pixelRatio = MediaQuery.devicePixelRatioOf(context)
```

- 设备像素比（Device Pixel Ratio）
- 表示物理像素与逻辑像素的比值
- 例如：Retina 屏幕的 pixelRatio 是 2.0 或 3.0

**为什么需要这些值**：

- `aspectRatio`：用于 `.sp` 计算（响应式字体大小）
- `pixelRatio`：用于 `.dp` 计算（密度无关像素）

### 第五步：设置设备类型

```dart 75:99:lib/util.dart
    // Sets DeviceType
    if (kIsWeb) {
      deviceType = DeviceType.web;
    } else {
      switch (defaultTargetPlatform) {
        case TargetPlatform.android:
          deviceType = DeviceType.android;
          break;
        case TargetPlatform.iOS:
          deviceType = DeviceType.ios;
          break;
        case TargetPlatform.windows:
          deviceType = DeviceType.windows;
          break;
        case TargetPlatform.macOS:
          deviceType = DeviceType.mac;
          break;
        case TargetPlatform.linux:
          deviceType = DeviceType.linux;
          break;
        case TargetPlatform.fuchsia:
          deviceType = DeviceType.fuchsia;
          break;
      }
    }
```

**判断逻辑**：

1. 首先检查是否为 Web 平台（`kIsWeb`）
2. 如果不是 Web，使用 `defaultTargetPlatform` 判断具体平台
3. 根据平台设置对应的 `DeviceType`

**平台检测的重要性**：

- 不同平台可能有不同的显示特性
- 某些功能可能只支持特定平台
- 可以根据平台调整 UI 或行为

### 第六步：设置屏幕类型

```dart 101:112:lib/util.dart
    // Sets ScreenType
    if ((orientation == Orientation.portrait && width <= maxMobileWidth) ||
        (orientation == Orientation.landscape && height <= maxMobileWidth)) {
      screenType = ScreenType.mobile;
    } else if (maxTabletWidth == null ||
        (orientation == Orientation.portrait && width <= maxTabletWidth) ||
        (orientation == Orientation.landscape && height <= maxTabletWidth)) {
      screenType = ScreenType.tablet;
    } else {
      screenType = ScreenType.desktop;
    }
```

这是屏幕类型判断的核心逻辑，需要仔细理解。

**判断逻辑详解**：

1. **移动设备判断**：

   ```dart
   (orientation == Orientation.portrait && width <= maxMobileWidth) ||
   (orientation == Orientation.landscape && height <= maxMobileWidth)
   ```

   - 竖屏时：如果宽度 ≤ maxMobileWidth，则为移动设备
   - 横屏时：如果高度 ≤ maxMobileWidth，则为移动设备
   - **为什么横屏时用高度？** 因为横屏时，原来的高度变成了宽度方向的尺寸

2. **平板设备判断**：

   ```dart
   maxTabletWidth == null ||
   (orientation == Orientation.portrait && width <= maxTabletWidth) ||
   (orientation == Orientation.landscape && height <= maxTabletWidth)
   ```

   - 如果 `maxTabletWidth` 为 null，则只区分移动和平板（没有桌面类型）
   - 否则，使用类似的逻辑判断是否为平板

3. **桌面设备判断**：

   ```dart
   else {
     screenType = ScreenType.desktop;
   }
   ```

   - 只有在设置了 `maxTabletWidth` 且不满足平板条件时，才是桌面设备

**示例说明**：

假设 `maxMobileWidth = 599`，`maxTabletWidth = 1024`：

- iPhone（竖屏，宽度 375）：`375 <= 599` → `mobile`
- iPad（竖屏，宽度 768）：`768 > 599` 且 `768 <= 1024` → `tablet`
- iPad Pro（竖屏，宽度 1024）：`1024 > 599` 且 `1024 <= 1024` → `tablet`
- 桌面显示器（宽度 1920）：`1920 > 599` 且 `1920 > 1024` → `desktop`

## BoxConstraints 详解

`BoxConstraints` 是 Flutter 布局系统的核心概念。让我们深入理解：

### BoxConstraints 的结构

```dart
class BoxConstraints {
  final double minWidth;
  final double maxWidth;
  final double minHeight;
  final double maxHeight;
}
```

### 在 Sizer 中的使用

Sizer 使用 `BoxConstraints` 的 `maxWidth` 和 `maxHeight` 作为屏幕尺寸。这是因为：

1. **LayoutBuilder 提供约束**：`Sizer` Widget 使用 `LayoutBuilder`，它提供的 `constraints` 包含了可用空间的信息
2. **全屏应用的约束**：对于全屏应用，`maxWidth` 和 `maxHeight` 就是屏幕尺寸
3. **嵌套布局的处理**：即使有嵌套的 `LayoutBuilder`，也能正确获取实际的可用空间

### constrainDimensions 方法

```dart
aspectRatio = constraints.constrainDimensions(width, height).aspectRatio
```

`constrainDimensions` 方法会根据约束调整尺寸，确保结果在约束范围内：

```dart
Size constrainDimensions(double width, double height) {
  return Size(
    width.clamp(minWidth, maxWidth),
    height.clamp(minHeight, maxHeight),
  );
}
```

然后计算调整后尺寸的宽高比。

## MediaQuery 的使用

Sizer 使用了 `MediaQuery` 的两个方法：

1. `MediaQuery.viewPaddingOf(context)`：获取视图内边距
2. `MediaQuery.devicePixelRatioOf(context)`：获取设备像素比

### 为什么使用静态方法？

代码中有注释显示从实例方法改为静态方法：

```dart
// final viewPadding = MediaQuery.of(context).viewPadding;
final viewPadding = MediaQuery.viewPaddingOf(context);
```

**静态方法的优势**：

- 不需要创建 `MediaQuery` 实例
- 更简洁的 API
- 性能稍好（避免了实例查找）

**使用场景**：

- `viewPaddingOf`：只需要视图内边距时使用
- `devicePixelRatioOf`：只需要像素比时使用
- `of(context)`：需要完整 MediaQuery 数据时使用

## 线程安全考虑

### 潜在问题

`Device` 类的静态变量在多线程环境下可能存在竞态条件。但在 Flutter 中：

1. **单线程模型**：Flutter 的 UI 操作都在主线程（UI 线程）执行
2. **Dart 的并发模型**：Dart 是单线程语言，通过事件循环处理异步
3. **初始化时机**：`setScreenSize` 在 Widget 构建时调用，此时已在主线程

因此，当前实现是安全的。

### 如果支持多线程会怎样？

如果将来 Flutter 支持真正的多线程，需要考虑：

- 使用锁机制保护静态变量
- 或者使用线程局部存储
- 或者重新设计为非静态结构

但目前的实现已经足够。

## 总结

`Device` 类是 Sizer 的核心，它：

1. **使用静态变量**：提供全局访问的设备信息
2. **延迟初始化**：使用 `late` 关键字，在 `setScreenSize` 中初始化
3. **全面收集信息**：收集尺寸、方向、类型、安全区域、像素密度等信息
4. **智能判断类型**：根据尺寸和方向判断设备类型和屏幕类型

理解 `Device` 类的实现，是掌握 Sizer 工作原理的关键。

## 检查清单

在进入下一章之前，确保你理解：

- [ ] 为什么 `Device` 类使用静态变量
- [ ] `late` 关键字的作用和使用场景
- [ ] `setScreenSize` 方法的完整流程
- [ ] SafeArea 的计算逻辑
- [ ] 屏幕类型判断算法的原理
- [ ] BoxConstraints 和 MediaQuery 的使用

## 实践练习

1. 分析屏幕类型判断算法，尝试不同的 `maxMobileWidth` 和 `maxTabletWidth` 值，看看结果如何变化。

2. 思考一下，如果需要在计算安全区域时考虑 AppBar 和 BottomNavigationBar，应该如何修改代码？

3. 尝试实现一个方法，根据设备类型返回不同的配置信息（如字体大小、间距等）。
