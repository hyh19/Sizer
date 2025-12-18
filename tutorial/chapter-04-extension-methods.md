# 第 4 章：扩展方法的计算算法

## 引言

扩展方法是 Sizer 最直观的 API，它们让开发者能够通过 `.h`、`.w`、`.sp`、`.dp` 等简洁的语法实现响应式布局。本章将深入分析每个扩展方法的数学原理和计算逻辑。

## 学习目标

- 理解 Dart 扩展方法的机制
- 掌握 `.h`、`.w` 等百分比计算的原理
- 深入理解 `.sp` 和 `.dp` 的密度计算算法
- 理解 `.vmin` 和 `.vmax` 的视口计算
- 掌握绝对单位转换的规则

## 扩展方法概述

让我们先看看 `SizerExt` 扩展的完整结构：

```dart 3:79:lib/extension.dart
extension SizerExt on num {
//  *****************  Absolute length units *****************************************
  // https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units

  /// The respective value in centimeters
  double get cm => this * 37.8;

  /// The respective value millimeters
  double get mm => this * 3.78;

  /// The respective value in quarter-millimeters
  double get Q => this * 0.945;

  /// The respective value in inches
  double get inches => this * 96;

  /// The respective value in picas (1/6th of 1 inch)
  double get pc => this * 16;

  /// The respective value in points (1/72th of 1 inch)
  double get pt => this * inches / 72;

  /// The respective value in pixels (default)
  double get px => this.toDouble();

  //  *****************  Relative length units *****************************************
  // https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units

  // TODO Recursive units need to be implemented
  /*double get em => ;
  double get ex => ;
  double get ch => ;
  double get rem => ;
  double get lh => ;*/

  /// Respective percentage of the viewport's smaller dimension.
  double get vmin => this * min(Device.height, Device.width) / 100;

  /// Respective percentage of the viewport's larger dimension.
  double get vmax => this * max(Device.height, Device.width) / 100;

  /// Calculates the height depending on the device's screen size
  ///
  /// Eg: 20.h -> will take 20% of the screen's height
  double get h => this * Device.height / 100;

  /// Calculates the width depending on the device's screen size
  ///
  /// Eg: 20.w -> will take 20% of the screen's width
  double get w => this * Device.width / 100;

  /// Calculates the height depending on the remaining device height
  /// after using `SafeArea`
  /// Eg: 20.sh -> will take 20% of the safe area height
  double get sh => this * Device.safeHeight / 100;

  /// Calculates the width depending on the remaining device width
  /// after using `SafeArea`
  ///
  /// Eg: 20.sw -> will take 20% of the safe area width
  double get sw => this * Device.safeWidth / 100;

  /// Calculates the sp (Scalable Pixel) depending on the device's pixel
  /// density and aspect ratio
  double get sp =>
      this *
      (((h + w) + (Device.pixelRatio * Device.aspectRatio)) / 2.08) /
      100;

  /// Calculates the material dp (Pixel Density)
  /// (https://material.io/design/layout/pixel-density.html#pixel-density-on-android))
  double get dp => this * (w * 160) / Device.pixelRatio;

  /// Calculates the sp (Scalable Pixel) based on Issue #27
  double get spa =>
      this * (((h + w) + (240 * Device.aspectRatio)) / 2.08) / 100;
}
```

扩展方法分为两大类：

1. **绝对长度单位**：cm、mm、inches 等物理单位
2. **相对长度单位**：基于屏幕尺寸的百分比单位

## Dart 扩展方法机制

### 扩展方法的基本语法

```dart
extension ExtensionName on Type {
  ReturnType get propertyName => /* ... */;
  ReturnType methodName() => /* ... */;
}
```

### 在 Sizer 中的应用

```dart
extension SizerExt on num {
  double get h => this * Device.height / 100;
}
```

**关键点**：

- `extension SizerExt on num`：为 `num` 类型（int 和 double 的父类）添加扩展
- `this`：指向调用扩展方法的数字实例
- 返回 `double`：确保精度和一致性

### 使用示例

```dart
50.h    // 相当于 50 * Device.height / 100
30.5.w  // 相当于 30.5 * Device.width / 100
```

**类型转换**：

- `50`（int）会自动转换为 `num`，然后调用扩展方法
- `30.5`（double）直接作为 `num` 调用扩展方法
- 返回的都是 `double` 类型

## 百分比计算：.h 和 .w

### .h 的实现

```dart 47:47:lib/extension.dart
  double get h => this * Device.height / 100;
```

**计算逻辑**：

```text
结果 = 输入值 × 屏幕高度 ÷ 100
```

**示例**：

假设屏幕高度为 800 像素：

- `20.h` = 20 × 800 ÷ 100 = 160 像素
- `50.h` = 50 × 800 ÷ 100 = 400 像素
- `100.h` = 100 × 800 ÷ 100 = 800 像素（全屏高度）

### .w 的实现

```dart 52:52:lib/extension.dart
  double get w => this * Device.width / 100;
```

**计算逻辑**：

```text
结果 = 输入值 × 屏幕宽度 ÷ 100
```

**示例**：

假设屏幕宽度为 400 像素：

- `25.w` = 25 × 400 ÷ 100 = 100 像素
- `50.w` = 50 × 400 ÷ 100 = 200 像素（半屏宽度）

### 使用场景

`.h` 和 `.w` 适用于：

- **响应式布局**：根据屏幕尺寸自动缩放
- **保持比例**：确保 UI 元素在不同设备上保持相同的视觉比例
- **简单计算**：直观的百分比计算

### 注意事项

1. **横竖屏切换**：横竖屏切换时，`.h` 和 `.w` 的值会自动更新
2. **精度**：使用 `double` 类型，保证计算精度
3. **性能**：直接读取静态变量，性能开销小

## 安全区域计算：.sh 和 .sw

### .sh 的实现

```dart 57:57:lib/extension.dart
  double get sh => this * Device.safeHeight / 100;
```

### .sw 的实现

```dart 63:63:lib/extension.dart
  double get sw => this * Device.safeWidth / 100;
```

### 与 .h 和 .w 的区别

- `.h`、`.w`：基于整个屏幕尺寸
- `.sh`、`.sw`：基于安全区域尺寸（排除状态栏、导航栏等）

**使用场景**：

- 需要在状态栏下方显示内容时，使用 `.sh`
- 需要在导航栏上方显示内容时，使用 `.sh`
- 需要避开屏幕缺口时，使用 `.sw`

**示例**：

```dart
// 使用安全区域高度，确保内容不会被状态栏遮挡
Container(
  height: 100.sh,  // 100% 的安全区域高度
  width: 50.sw,    // 50% 的安全区域宽度
)
```

## 视口单位：.vmin 和 .vmax

### .vmin 的实现

```dart 39:39:lib/extension.dart
  double get vmin => this * min(Device.height, Device.width) / 100;
```

**计算逻辑**：

```text
vmin = 输入值 × min(高度, 宽度) ÷ 100
```

`vmin` 基于屏幕较小的那个维度。

### .vmax 的实现

```dart 42:42:lib/extension.dart
  double get vmax => this * max(Device.height, Device.width) / 100;
```

**计算逻辑**：

```text
vmax = 输入值 × max(高度, 宽度) ÷ 100
```

`vmax` 基于屏幕较大的那个维度。

### 使用场景

`vmin` 和 `vmax` 适用于：

- **正方形元素**：需要保持正方形，不受屏幕方向影响
- **圆形元素**：需要保持圆形，使用 `vmin`
- **响应式字体**：根据屏幕尺寸调整字体大小

**示例**：

```dart
// 创建一个正方形，边长始终是屏幕较小维度的 30%
Container(
  width: 30.vmin,
  height: 30.vmin,
  decoration: BoxDecoration(shape: BoxShape.circle),
)
```

## 字体大小计算：.sp 和 .spa

### .sp 的实现

这是最复杂的计算方法之一：

```dart 67:70:lib/extension.dart
  double get sp =>
      this *
      (((h + w) + (Device.pixelRatio * Device.aspectRatio)) / 2.08) /
      100;
```

**计算逻辑详解**：

让我们逐步分解这个公式：

1. **计算基础尺寸**：`h + w`
   - 这是高度和宽度的总和
   - 提供了一个综合的屏幕尺寸指标

2. **计算密度因子**：`Device.pixelRatio * Device.aspectRatio`
   - `pixelRatio`：设备像素比
   - `aspectRatio`：屏幕宽高比
   - 这个因子考虑了设备的显示特性

3. **综合计算**：`(h + w + 密度因子) / 2.08`
   - 将基础尺寸和密度因子相加
   - 除以 2.08（经验常数）进行缩放

4. **百分比转换**：`输入值 × 综合值 / 100`

**公式的数学意义**：

```text
sp = 输入值 × (高度 + 宽度 + 像素比 × 宽高比) / 2.08 / 100
```

这个公式试图：

- 考虑屏幕的整体尺寸（高度 + 宽度）
- 考虑设备的显示密度（像素比）
- 考虑屏幕的形状（宽高比）
- 通过常数 2.08 进行校准

### .spa 的实现

```dart 77:78:lib/extension.dart
  double get spa =>
      this * (((h + w) + (240 * Device.aspectRatio)) / 2.08) / 100;
```

`.spa` 是 `.sp` 的变体，将 `Device.pixelRatio` 替换为固定值 240。

**设计原因**：

根据注释，这是基于 Issue #27 的解决方案。可能的原因：

- `pixelRatio` 在某些设备上可能不准确
- 使用固定值可以提供更一致的体验
- 简化计算逻辑

### 使用场景

`.sp` 和 `.spa` 主要用于字体大小：

```dart
Text(
  'Hello',
  style: TextStyle(fontSize: 16.sp),  // 响应式字体大小
)
```

**对比**：

- `.sp`：考虑设备的实际像素密度
- `.spa`：使用固定的密度值，可能在某些设备上更稳定

## 密度无关像素：.dp

### .dp 的实现

```dart 74:74:lib/extension.dart
  double get dp => this * (w * 160) / Device.pixelRatio;
```

### 计算公式分析

```text
dp = 输入值 × (宽度 × 160) / 像素比
```

**数学推导**：

Material Design 的 dp 定义基于：

```text
dp = px / (dpi / 160)
```

其中：

- `px`：物理像素
- `dpi`：每英寸点数
- `160`：基准密度（mdpi）

在 Sizer 的实现中：

1. `w * 160`：将逻辑宽度（像素）转换为基准密度下的宽度
2. `/ Device.pixelRatio`：除以像素比，得到逻辑像素

**示例**：

假设：

- 屏幕宽度：400 像素
- 像素比：2.0

计算 `10.dp`：

```text
10.dp = 10 × (400 × 160) / 2.0 / 100
      = 10 × 64000 / 2.0 / 100
      = 10 × 320 / 100
      = 32 像素
```

### 与 Material Design 的关系

`.dp` 试图实现 Material Design 的密度无关像素概念，但实现方式有所不同：

- **Material Design**：基于物理尺寸（英寸）和 DPI
- **Sizer**：基于逻辑宽度和像素比

这种实现方式可能在某些设备上略有差异，但对于大多数场景已经足够。

### 使用场景

`.dp` 主要用于：

- **Material Design 风格的应用**：遵循 Material Design 规范
- **需要与 Android 原生一致的场景**：与 Android 的 dp 单位对应

## 绝对单位转换

Sizer 还提供了一系列绝对单位的转换：

### 厘米（.cm）

```dart 8:8:lib/extension.dart
  double get cm => this * 37.8;
```

**转换规则**：1 厘米 = 37.8 像素

这是基于 96 DPI（每英寸 96 像素）的标准：

```text
1 英寸 = 96 像素
1 厘米 = 1 英寸 / 2.54 ≈ 37.8 像素
```

### 毫米（.mm）

```dart 11:11:lib/extension.dart
  double get mm => this * 3.78;
```

**转换规则**：1 毫米 = 3.78 像素

```text
1 毫米 = 1 厘米 / 10 = 37.8 / 10 = 3.78 像素
```

### 四分之一毫米（.Q）

```dart 14:14:lib/extension.dart
  double get Q => this * 0.945;
```

**转换规则**：1 Q = 0.945 像素

```text
1 Q = 1 毫米 / 4 = 3.78 / 4 = 0.945 像素
```

### 英寸（.inches）

```dart 17:17:lib/extension.dart
  double get inches => this * 96;
```

**转换规则**：1 英寸 = 96 像素

这是 CSS 的标准定义。

### 派卡（.pc）

```dart 20:20:lib/extension.dart
  double get pc => this * 16;
```

**转换规则**：1 派卡 = 16 像素

```text
1 派卡 = 1 英寸 / 6 = 96 / 6 = 16 像素
```

### 点（.pt）

```dart 23:23:lib/extension.dart
  double get pt => this * inches / 72;
```

**转换规则**：1 点 = 1 英寸 / 72 = 96 / 72 ≈ 1.333 像素

这是排版中常用的单位。

### 像素（.px）

```dart 26:26:lib/extension.dart
  double get px => this.toDouble();
```

**转换规则**：1 像素 = 1 像素（无转换）

`.px` 只是将数字转换为 `double`，不进行任何计算。

### 单位转换关系图

```text
英寸 (96px)
├── 派卡 (16px = 1/6 英寸)
└── 点 (1.333px = 1/72 英寸)

厘米 (37.8px = 1/2.54 英寸)
└── 毫米 (3.78px)
    └── 四分之一毫米 (0.945px)
```

## 未实现的单位

代码中注释了未实现的相对单位：

```dart 31:36:lib/extension.dart
  // TODO Recursive units need to be implemented
  /*double get em => ;
  double get ex => ;
  double get ch => ;
  double get rem => ;
  double get lh => ;*/
```

这些是 CSS 中的相对单位：

- `em`：相对于当前元素的字体大小
- `ex`：相对于字符 "x" 的高度
- `ch`：相对于字符 "0" 的宽度
- `rem`：相对于根元素的字体大小
- `lh`：相对于行高

**为什么未实现**：

这些单位需要上下文信息（如当前字体大小），在 Flutter 的扩展方法中难以实现。如果需要这些功能，可能需要不同的实现方式。

## 性能考虑

### 计算开销

扩展方法的计算都很简单，主要是：

- 乘法运算
- 除法运算
- 读取静态变量

这些操作的开销都很小。

### 缓存策略

`Device` 类的静态变量在 `setScreenSize` 时初始化，后续直接读取，无需重复计算。

### 最佳实践

1. **避免重复计算**：如果需要多次使用相同的值，先计算并存储
2. **选择合适的单位**：根据场景选择合适的单位，避免不必要的转换
3. **注意精度**：使用 `double` 类型，注意浮点数精度问题

## 总结

Sizer 的扩展方法提供了丰富的响应式计算方式：

- **百分比单位**：`.h`、`.w`、`.sh`、`.sw` 用于布局
- **视口单位**：`.vmin`、`.vmax` 用于保持形状
- **字体单位**：`.sp`、`.spa`、`.dp` 用于文字大小
- **绝对单位**：cm、mm、inches 等用于精确尺寸

每个单位都有其特定的使用场景和计算逻辑。

## 检查清单

在进入下一章之前，确保你理解：

- [ ] Dart 扩展方法的机制
- [ ] `.h` 和 `.w` 的百分比计算原理
- [ ] `.sp` 计算公式的每个组成部分
- [ ] `.dp` 与 Material Design 的关系
- [ ] `.vmin` 和 `.vmax` 的区别和使用场景
- [ ] 绝对单位的转换规则

## 实践练习

1. 手动计算几个 `.sp` 的值，验证你对公式的理解。

2. 对比 `.sp` 和 `.spa` 在不同设备上的表现，分析差异。

3. 实现一个自定义的扩展方法，提供不同的字体大小计算方式。

4. 思考一下，如果要实现 `em` 单位，需要如何修改 Sizer 的架构？
