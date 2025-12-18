# 第 6 章：Adaptive 静态类的作用

## 引言

`Adaptive` 类提供了静态方法版本的响应式计算，作为扩展方法的替代方案。本章将深入分析 `Adaptive` 类的设计意图、实现方式和使用场景。

## 学习目标

- 理解 `Adaptive` 类的设计目的
- 掌握 `Adaptive` 类与扩展方法的关系
- 理解何时使用 `Adaptive` 方法，何时使用扩展方法
- 理解代码复用和封装策略

## Adaptive 类概览

让我们先看看 `Adaptive` 类的完整代码：

```dart 115:164:lib/util.dart
class Adaptive {
  /// Calculates the height depending on the device's screen size
  ///
  /// Eg: 20.h -> will take 20% of the screen's height
  static double h(num height) => height.h;

  /// Calculates the width depending on the device's screen size
  ///
  /// Eg: 20.w -> will take 20% of the screen's width
  static double w(num width) => width.w;

  /// Calculates the height depending on the device's screen size
  ///
  /// Eg: 20.sh -> will take 20% of the safe area height
  static double sh(num height) => height.sh;

  /// Calculates the width depending on the device's screen size
  ///
  /// Eg: 20.sw -> will take 20% of the safe area width
  static double sw(num width) => width.sw;

  /// Calculates the sp (Scalable Pixel) depending on the device's pixel
  /// density and aspect ratio
  static double sp(num scalablePixel) => scalablePixel.sp;

  /// Calculates the material dp (Pixel Density)
  /// (https://material.io/design/layout/pixel-density.html#pixel-density-on-android))
  static double dp(num densityPixel) => densityPixel.dp;

  /// The respective value in centimeters
  static double cm(num centimeters) => centimeters.cm;

  /// The respective value in millimeters
  static double mm(num millimeters) => millimeters.mm;

  /// The respective value in quarter-millimeters
  static double Q(num quarterMillimeters) => quarterMillimeters.Q;

  /// The respective value in inches
  static double inches(num inches) => inches.inches;

  /// The respective value in picas (1/6th of 1 inch)
  static double pc(num picas) => picas.pc;

  /// The respective value in points (1/72th of 1 inch)
  static double pt(num point) => point.pt;

  /// The respective value in pixels (default)
  static double px(num pixels) => pixels.px;
}
```

## 设计模式：适配器模式

`Adaptive` 类采用了适配器模式（Adapter Pattern）：

- **目标接口**：静态方法接口
- **适配对象**：扩展方法
- **适配器**：`Adaptive` 类

这种设计将扩展方法的接口适配为静态方法接口。

## 功能对应关系

`Adaptive` 类的每个静态方法都对应一个扩展方法：

| Adaptive 静态方法 | 扩展方法 | 功能 |
|------------------|---------|------|
| `Adaptive.h(num)` | `num.h` | 基于屏幕高度的百分比 |
| `Adaptive.w(num)` | `num.w` | 基于屏幕宽度的百分比 |
| `Adaptive.sh(num)` | `num.sh` | 基于安全区域高度的百分比 |
| `Adaptive.sw(num)` | `num.sw` | 基于安全区域宽度的百分比 |
| `Adaptive.sp(num)` | `num.sp` | 可缩放像素 |
| `Adaptive.dp(num)` | `num.dp` | 密度无关像素 |
| `Adaptive.cm(num)` | `num.cm` | 厘米 |
| `Adaptive.mm(num)` | `num.mm` | 毫米 |
| `Adaptive.Q(num)` | `num.Q` | 四分之一毫米 |
| `Adaptive.inches(num)` | `num.inches` | 英寸 |
| `Adaptive.pc(num)` | `num.pc` | 派卡 |
| `Adaptive.pt(num)` | `num.pt` | 点 |
| `Adaptive.px(num)` | `num.px` | 像素 |

## 实现方式：委托模式

每个 `Adaptive` 静态方法都直接调用对应的扩展方法：

```dart
static double h(num height) => height.h;
```

这是委托模式（Delegate Pattern）的典型应用：

- `Adaptive.h` 不执行实际计算
- 委托给 `height.h` 扩展方法
- 实现代码复用，避免重复实现

## 代码复用策略

### 避免重复实现

如果 `Adaptive` 类不委托给扩展方法，需要重复实现所有计算逻辑：

```dart
// 不好的实现（重复代码）
class Adaptive {
  static double h(num height) {
    return height * Device.height / 100;  // 重复实现
  }
  
  static double w(num width) {
    return width * Device.width / 100;    // 重复实现
  }
  // ... 更多重复代码
}
```

### 单一数据源

通过委托，确保计算逻辑只有一份：

- 扩展方法：实际的实现
- `Adaptive` 类：调用扩展方法

这样，如果需要修改计算逻辑，只需要修改扩展方法一处。

## 使用场景对比

### 扩展方法的使用

扩展方法更符合 Dart 的语法习惯：

```dart
Container(
  width: 50.w,     // 简洁直观
  height: 30.h,
)
```

**优势**：

- 语法简洁
- 链式调用方便
- 符合 Dart 的代码风格

**限制**：

- 需要导入 `sizer` 包
- 某些情况下可能不够明确

### Adaptive 静态方法的使用

静态方法在某些场景下更清晰：

```dart
Container(
  width: Adaptive.w(50),    // 明确表示是响应式计算
  height: Adaptive.h(30),
)
```

**优势**：

- 明确表示这是响应式计算
- 函数式风格
- 在某些 IDE 中自动补全更好

**劣势**：

- 代码更长
- 不够简洁

## 实际使用建议

### 推荐使用扩展方法

在大多数情况下，推荐使用扩展方法：

```dart
// 推荐
Container(
  width: 50.w,
  height: 30.h,
  child: Text('Hello', style: TextStyle(fontSize: 16.sp)),
)
```

**原因**：

- 代码更简洁
- 更符合 Dart 的习惯用法
- 可读性更好

### 使用 Adaptive 的场景

在某些特定场景下，`Adaptive` 静态方法可能更合适：

#### 1. 函数式编程风格

```dart
final width = Adaptive.w(50);
final height = Adaptive.h(30);
```

#### 2. 动态计算

```dart
double calculateWidth(double percentage) {
  return Adaptive.w(percentage);  // 使用变量，扩展方法不方便
}
```

#### 3. 代码可读性

某些开发者可能更喜欢静态方法的明确性：

```dart
// 有些人认为这样更清晰
Adaptive.w(50)  // 明确表示这是 Adaptive 计算
// vs
50.w            // 需要知道这是扩展方法
```

#### 4. IDE 支持

在某些 IDE 中，静态方法的自动补全可能更好：

```dart
Adaptive.  // IDE 会提示所有可用方法
```

## 性能考虑

### 性能差异

两种方式的性能几乎相同：

```dart
50.w              // 扩展方法
Adaptive.w(50)    // 静态方法 -> 扩展方法
```

因为 `Adaptive` 方法直接调用扩展方法，没有额外的开销：

```dart
static double w(num width) => width.w;  // 只是一个方法调用
```

### 编译优化

Dart 编译器可能会内联这些简单的方法调用，使得性能差异可以忽略不计。

## 设计决策分析

### 为什么提供两种方式？

1. **灵活性**：满足不同开发者的偏好
2. **兼容性**：某些场景下静态方法更方便
3. **明确性**：静态方法更明确表示这是响应式计算

### 为什么不只提供一种方式？

如果只提供扩展方法：

- 某些场景下不够灵活
- 函数式编程风格的支持不够好

如果只提供静态方法：

- 代码不够简洁
- 不符合 Dart 的习惯用法

提供两种方式，让开发者可以根据场景和偏好选择。

## 代码组织

### 位置选择

`Adaptive` 类放在 `util.dart` 文件中，与 `Device` 类在一起：

- `Device` 类：设备信息管理
- `Adaptive` 类：响应式计算接口

这种组织方式逻辑清晰。

### 命名约定

`Adaptive` 这个名字表达了：

- **适应性**：能够适应不同屏幕尺寸
- **响应式**：响应设备变化

这是一个合适的选择。

## 扩展方法 vs 静态方法

### 技术对比

| 特性 | 扩展方法 | 静态方法 |
|------|---------|---------|
| 语法 | `50.w` | `Adaptive.w(50)` |
| 代码长度 | 短 | 长 |
| 可读性 | 简洁 | 明确 |
| 函数式风格 | 支持 | 更好 |
| IDE 支持 | 一般 | 更好 |
| 性能 | 相同 | 相同 |

### 选择建议

**优先使用扩展方法**，除非：

1. 需要使用变量或表达式
2. 偏好函数式编程风格
3. IDE 对静态方法的支持明显更好

## 总结

`Adaptive` 类作为扩展方法的静态方法接口：

1. **采用适配器模式**：将扩展方法接口适配为静态方法接口
2. **使用委托模式**：所有方法都委托给扩展方法，避免重复实现
3. **提供灵活性**：满足不同开发者的偏好和使用场景
4. **保持一致性**：两种方式功能完全相同，只是接口不同

理解 `Adaptive` 类的设计，有助于更好地使用 Sizer 库，并在需要时选择合适的接口。

## 检查清单

在进入下一章之前，确保你理解：

- [ ] `Adaptive` 类的设计目的和作用
- [ ] `Adaptive` 类与扩展方法的关系
- [ ] 两种使用方式的区别和选择
- [ ] 代码复用策略和设计模式
- [ ] 性能和实现细节

## 实践练习

1. 尝试在同一个项目中同时使用扩展方法和 `Adaptive` 静态方法，体验两种方式的差异。

2. 实现一个工具函数，使用 `Adaptive` 方法计算响应式尺寸。

3. 思考一下，如果 Sizer 只提供扩展方法或只提供静态方法，会有什么影响？

4. 分析一下，是否还有其他场景适合使用 `Adaptive` 静态方法而不是扩展方法？
