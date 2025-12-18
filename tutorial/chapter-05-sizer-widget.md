# 第 5 章：Sizer Widget 的实现机制

## 引言

`Sizer` Widget 是 Sizer 库的入口点，它负责初始化设备信息并响应布局和方向变化。理解 `Sizer` Widget 的实现，是掌握整个库工作机制的关键。

## 学习目标

- 理解 `Sizer` Widget 的完整实现
- 掌握 `LayoutBuilder` 和 `OrientationBuilder` 的组合使用
- 理解断点系统（maxMobileWidth、maxTabletWidth）的实现逻辑
- 理解响应式构建的触发机制
- 掌握边界情况的处理

## Sizer Widget 概览

让我们先看看 `Sizer` Widget 的完整代码：

```dart 13:61:lib/widget.dart
/// A widget that gets the device's details like orientation and constraints
///
/// Usage: Wrap MaterialApp with this widget
class Sizer extends StatelessWidget {
  const Sizer({
    Key? key,
    required this.builder,
    this.maxMobileWidth = 599,
    this.maxTabletWidth,
  }) : super(key: key);

  /// Builds the widget whenever the orientation changes
  final ResponsiveBuilderType builder;

  /// This is the breakpoint used to determine whether the device is
  /// a mobile device or a tablet.
  ///
  /// If the `MediaQuery`'s width **in portrait mode** is less than or equal
  /// to `maxMobileWidth`, the device is in a mobile device
  final double maxMobileWidth;

  /// By default, the `ScreenType` can only be mobile or tablet. If this is set,
  /// the `ScreenType` can be desktop as well
  ///
  /// This is the breakpoint used to determine whether the device is
  /// a tablet or a desktop.
  ///
  /// If the `MediaQuery`'s width **in portrait mode** is
  /// less than or equal to `maxTabletWidth`, the device is in a tablet device
  final double? maxTabletWidth;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
      return OrientationBuilder(builder: (context, orientation) {
        Device.setScreenSize(
          context,
          constraints,
          orientation,
          maxMobileWidth,
          maxTabletWidth,
        );

        if (constraints.maxWidth == 0 || constraints.maxHeight == 0) {
          return const SizedBox();
        }
        return builder(context, orientation, Device.screenType);
      });
    });
  }
}
```

## ResponsiveBuilderType 类型定义

在分析 `Sizer` Widget 之前，我们先看看构建器函数的类型定义：

```dart 4:8:lib/widget.dart
/// Provides `Context`, `Orientation`, and `ScreenType` parameters to the builder function
typedef ResponsiveBuilderType = Widget Function(
  BuildContext,
  Orientation,
  ScreenType,
);
```

这是一个类型别名，定义了构建器函数的签名：

- **参数 1**：`BuildContext` - 构建上下文
- **参数 2**：`Orientation` - 屏幕方向（竖屏或横屏）
- **参数 3**：`ScreenType` - 屏幕类型（移动设备、平板或桌面）
- **返回值**：`Widget` - 构建的 Widget

## Sizer 类的结构

### 类定义

```dart 13:19:lib/widget.dart
class Sizer extends StatelessWidget {
  const Sizer({
    Key? key,
    required this.builder,
    this.maxMobileWidth = 599,
    this.maxTabletWidth,
  }) : super(key: key);
```

`Sizer` 继承自 `StatelessWidget`，这意味着：

- 它是无状态的 Widget
- 不会维护内部状态
- 依赖外部传入的参数进行构建

### 构造函数参数

1. **`builder`**（必需）：
   - 类型：`ResponsiveBuilderType`
   - 用途：构建实际的 Widget 树

2. **`maxMobileWidth`**（可选，默认 599）：
   - 类型：`double`
   - 用途：区分移动设备和平板设备的断点

3. **`maxTabletWidth`**（可选）：
   - 类型：`double?`
   - 用途：区分平板设备和桌面设备的断点
   - 如果为 `null`，则只区分移动设备和平板设备

## build 方法的实现

`build` 方法是 `Sizer` Widget 的核心，让我们逐步分析：

### LayoutBuilder 的使用

```dart 42:43:lib/widget.dart
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
```

`LayoutBuilder` 是 Flutter 提供的 Widget，它：

- 提供 `BoxConstraints` 信息
- 在布局约束变化时重建
- 不需要 `MediaQuery`，直接获取约束信息

**为什么使用 LayoutBuilder**：

- **直接获取约束**：不需要通过 `MediaQuery` 获取尺寸
- **精确的布局信息**：约束信息比 `MediaQuery` 的尺寸更精确
- **性能更好**：只在约束变化时重建

### OrientationBuilder 的使用

```dart 43:44:lib/widget.dart
      return OrientationBuilder(builder: (context, orientation) {
```

`OrientationBuilder` 是另一个 Flutter 提供的 Widget，它：

- 提供当前的屏幕方向（`Orientation.portrait` 或 `Orientation.landscape`）
- 在方向变化时重建

**为什么嵌套使用**：

- `LayoutBuilder` 提供约束信息
- `OrientationBuilder` 提供方向信息
- 两者结合，可以同时响应尺寸和方向变化

### 设备信息初始化

```dart 45:51:lib/widget.dart
        Device.setScreenSize(
          context,
          constraints,
          orientation,
          maxMobileWidth,
          maxTabletWidth,
        );
```

这里调用 `Device.setScreenSize` 初始化所有设备信息：

1. 传入 `context`：用于访问 `MediaQuery`
2. 传入 `constraints`：来自 `LayoutBuilder`
3. 传入 `orientation`：来自 `OrientationBuilder`
4. 传入断点参数：用于判断屏幕类型

**调用时机**：

每次 `LayoutBuilder` 或 `OrientationBuilder` 触发重建时，都会重新初始化设备信息。这确保了：

- 设备信息始终是最新的
- 响应式计算基于最新的屏幕状态
- 横竖屏切换时信息及时更新

### 边界情况处理

```dart 53:55:lib/widget.dart
        if (constraints.maxWidth == 0 || constraints.maxHeight == 0) {
          return const SizedBox();
        }
```

这是一个重要的边界情况处理：

**为什么需要这个检查**：

在某些情况下，`LayoutBuilder` 可能会在布局计算期间提供无效的约束：

- `maxWidth == 0` 或 `maxHeight == 0` 表示尺寸还未确定
- 此时构建 Widget 可能导致布局错误
- 返回 `SizedBox()`（空 Widget）可以安全地处理这种情况

**何时会发生**：

- Widget 树初始构建时
- 某些动画或过渡期间
- 窗口大小调整时（桌面应用）

**处理方式**：

返回 `const SizedBox()`，这是一个不占用空间的空 Widget，不会影响布局。

### 构建最终 Widget

```dart 56:56:lib/widget.dart
        return builder(context, orientation, Device.screenType);
```

调用用户提供的 `builder` 函数，传入：

- `context`：构建上下文
- `orientation`：当前屏幕方向
- `Device.screenType`：当前屏幕类型（已由 `setScreenSize` 计算）

用户可以使用这些信息构建响应式 UI。

## 响应式更新机制

### 更新触发条件

`Sizer` Widget 会在以下情况触发重建：

1. **屏幕尺寸变化**：
   - `LayoutBuilder` 检测到约束变化
   - 例如：窗口大小调整、设备旋转（导致宽度和高度交换）

2. **屏幕方向变化**：
   - `OrientationBuilder` 检测到方向变化
   - 例如：从竖屏旋转到横屏

3. **父 Widget 重建**：
   - 如果父 Widget 重建，`Sizer` 也会重建
   - 但 `LayoutBuilder` 和 `OrientationBuilder` 会判断是否需要真正重建子 Widget

### 更新流程

```text
1. 屏幕变化（尺寸或方向）
   ↓
2. LayoutBuilder/OrientationBuilder 检测到变化
   ↓
3. Sizer.build 被调用
   ↓
4. Device.setScreenSize 更新设备信息
   ↓
5. builder 函数被调用，重建 UI
   ↓
6. 扩展方法使用新的设备信息计算尺寸
```

### 性能考虑

**优化点**：

1. **静态变量缓存**：设备信息存储在静态变量中，避免重复计算
2. **条件重建**：`LayoutBuilder` 和 `OrientationBuilder` 只在真正变化时重建
3. **常量 Widget**：边界情况处理使用 `const SizedBox()`

**潜在问题**：

- 每次重建都会调用 `setScreenSize`，但这个方法本身开销很小
- 如果 `builder` 函数很复杂，可能会导致性能问题

## 断点系统详解

### maxMobileWidth

```dart 24:29:lib/widget.dart
  /// This is the breakpoint used to determine whether the device is
  /// a mobile device or a tablet.
  ///
  /// If the `MediaQuery`'s width **in portrait mode** is less than or equal
  /// to `maxMobileWidth`, the device is in a mobile device
  final double maxMobileWidth;
```

**默认值**：599

**判断逻辑**（在 `Device.setScreenSize` 中）：

```dart
if ((orientation == Orientation.portrait && width <= maxMobileWidth) ||
    (orientation == Orientation.landscape && height <= maxMobileWidth)) {
  screenType = ScreenType.mobile;
}
```

- 竖屏时：宽度 ≤ 599 → 移动设备
- 横屏时：高度 ≤ 599 → 移动设备（因为横屏时高度对应竖屏的宽度）

### maxTabletWidth

```dart 31:39:lib/widget.dart
  /// By default, the `ScreenType` can only be mobile or tablet. If this is set,
  /// the `ScreenType` can be desktop as well
  ///
  /// This is the breakpoint used to determine whether the device is
  /// a tablet or a desktop.
  ///
  /// If the `MediaQuery`'s width **in portrait mode** is
  /// less than or equal to `maxTabletWidth`, the device is in a tablet device
  final double? maxTabletWidth;
```

**默认值**：`null`（可选）

**判断逻辑**（在 `Device.setScreenSize` 中）：

```dart
else if (maxTabletWidth == null ||
    (orientation == Orientation.portrait && width <= maxTabletWidth) ||
    (orientation == Orientation.landscape && height <= maxTabletWidth)) {
  screenType = ScreenType.tablet;
} else {
  screenType = ScreenType.desktop;
}
```

- 如果为 `null`：只区分移动设备和平板设备
- 如果设置：宽度/高度 ≤ maxTabletWidth → 平板设备，否则 → 桌面设备

### 断点选择建议

**移动应用**：

```dart
Sizer(
  maxMobileWidth: 599,  // 默认值，通常不需要修改
  builder: (context, orientation, screenType) {
    // ...
  },
)
```

**支持桌面应用**：

```dart
Sizer(
  maxMobileWidth: 599,
  maxTabletWidth: 1024,  // 设置桌面断点
  builder: (context, orientation, screenType) {
    // 现在 screenType 可能是 mobile、tablet 或 desktop
  },
)
```

## 使用示例

### 基本使用

```dart
Sizer(
  builder: (context, orientation, screenType) {
    return MaterialApp(
      home: HomePage(),
    );
  },
)
```

### 自定义断点

```dart
Sizer(
  maxMobileWidth: 600,
  maxTabletWidth: 1200,
  builder: (context, orientation, screenType) {
    return MaterialApp(
      home: ResponsiveHomePage(),
    );
  },
)
```

### 响应式 UI

```dart
Sizer(
  builder: (context, orientation, screenType) {
    return MaterialApp(
      home: screenType == ScreenType.tablet
        ? TabletLayout()
        : MobileLayout(),
    );
  },
)
```

## 与 Flutter Widget 系统的集成

### Widget 树位置

`Sizer` 应该放在 Widget 树的根部，通常包裹 `MaterialApp`：

```dart
void main() {
  runApp(
    Sizer(
      builder: (context, orientation, screenType) {
        return MaterialApp(
          home: HomePage(),
        );
      },
    ),
  );
}
```

**为什么放在根部**：

- 需要访问全局的布局信息
- 需要在方向变化时重建整个应用
- 需要为所有子 Widget 提供设备信息

### 与 MediaQuery 的关系

`Sizer` 内部使用 `MediaQuery` 获取一些信息（如 `viewPadding` 和 `devicePixelRatio`），但主要依赖 `LayoutBuilder` 和 `OrientationBuilder`。

**优势**：

- 不依赖 `MediaQuery` 的尺寸信息，更精确
- 可以在 `MediaQuery` 不可用时工作
- 性能更好

## 最佳实践

### 1. 合理设置断点

根据你的应用需求设置断点：

- 移动应用：使用默认值即可
- 支持平板：考虑调整 `maxMobileWidth`
- 支持桌面：设置 `maxTabletWidth`

### 2. 优化 builder 函数

`builder` 函数会在每次重建时调用，应该：

- 避免在 `builder` 中进行重计算
- 使用 `const` Widget 减少重建
- 合理使用 `StatefulWidget` 管理状态

### 3. 处理边界情况

虽然 `Sizer` 已经处理了 `constraints` 为 0 的情况，但在 `builder` 中仍应：

- 检查 `orientation` 和 `screenType` 的有效性
- 处理极端尺寸（如非常小的窗口）
- 提供降级方案

## 总结

`Sizer` Widget 通过 `LayoutBuilder` 和 `OrientationBuilder` 的组合使用，实现了：

1. **自动检测变化**：尺寸和方向变化时自动重建
2. **设备信息初始化**：每次重建时更新设备信息
3. **断点系统**：通过 `maxMobileWidth` 和 `maxTabletWidth` 判断屏幕类型
4. **边界情况处理**：安全处理无效约束

理解 `Sizer` Widget 的实现，是使用和扩展 Sizer 库的基础。

## 检查清单

在进入下一章之前，确保你理解：

- [ ] `Sizer` Widget 的完整结构和参数
- [ ] `LayoutBuilder` 和 `OrientationBuilder` 的作用和组合使用
- [ ] 设备信息初始化的时机和过程
- [ ] 边界情况处理的原因和方式
- [ ] 断点系统的判断逻辑
- [ ] 响应式更新的触发机制

## 实践练习

1. 尝试修改 `maxMobileWidth` 和 `maxTabletWidth` 的值，观察 `Device.screenType` 的变化。

2. 实现一个自定义的响应式 Widget，根据 `screenType` 显示不同的布局。

3. 分析一下，如果需要在 `Sizer` 中添加新的功能（如检测暗色模式），应该如何修改代码？

4. 思考一下，`Sizer` 的 `builder` 函数中可以访问 `MediaQuery` 吗？为什么？
