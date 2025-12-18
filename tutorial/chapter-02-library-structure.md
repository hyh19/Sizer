# 第 2 章：库结构与 part "../sizer-source-code-tutorial"机制

## 引言

Sizer 库使用了 Dart 的 `part"../sizer-source-code-tutorial"` 和 `part of"../sizer-source-code-tutorial"` 机制来组织代码。这种组织方式与常见的独立文件导入方式不同，有其特定的用途和优势。本章将深入解析这种代码组织方式。

## 学习目标

- 理解 Dart 的 `part"../sizer-source-code-tutorial"` 和 `part of"../sizer-source-code-tutorial"` 机制
- 掌握 Sizer 的文件组织结构
- 理解为什么选择 `part"../sizer-source-code-tutorial"` 而非独立文件
- 了解这种组织方式的优缺点

## 源码结构概览

让我们先看看 Sizer 的主库文件：

```dart 1:17:lib/sizer.dart
/*
 * Created by Prashant Padmani on 2018/9/29.
 * email: prashant09mca@gmail.com
*/
library sizer;

import 'dart:math';

import 'package:flutter/widgets.dart';
import 'package:flutter/foundation.dart' show defaultTargetPlatform, kIsWeb;

part '../sizer-source-code-tutorial/extension.dart';

part '../sizer-source-code-tutorial/util.dart';

part '../sizer-source-code-tutorial/widget.dart';
```

这个文件非常简洁，只包含：

1. 库声明：`library sizer;`
2. 导入语句：外部依赖的导入
3. Part 声明：声明包含的其他文件

## Dart 的 part "../sizer-source-code-tutorial"机制详解

### part "../sizer-source-code-tutorial"和 part of 的基本概念

在 Dart 中，`part"../sizer-source-code-tutorial"` 和 `part of"../sizer-source-code-tutorial"` 是一种将单个库拆分成多个文件的方式。它们的关系是：

- **主文件**：使用 `library` 声明库名，使用 `part"../sizer-source-code-tutorial"` 声明包含的文件
- **Part 文件**：使用 `part of"../sizer-source-code-tutorial"` 声明所属的库名

### part "../sizer-source-code-tutorial"的工作机制

当我们使用 `part"../sizer-source-code-tutorial"` 时：

1. **编译时合并**：所有 `part"../sizer-source-code-tutorial"` 文件在编译时会被合并到主库中
2. **共享作用域**：所有 `part"../sizer-source-code-tutorial"` 文件共享主库的导入和作用域
3. **单一库单元**：对外部来说，这是一个完整的库单元

### 示例说明

假设我们有这样的结构：

```dart
// main.dart
library my_lib;
import 'package:some_package/some_package.dart';

part '../sizer-source-code-tutorial/helper.dart';

class MainClass {
  void method() {
    HelperClass().doSomething(); // 可以直接使用
  }
}

// helper.dart
part of my_lib;  // 注意：没有 import

class HelperClass {
  void doSomething() {
    // 可以直接使用 some_package 的内容
    // 不需要再次导入
  }
}
```

关键点：

- `helper.dart` 使用 `part of my_lib`，表示它是 `my_lib` 的一部分
- `helper.dart` 不需要单独导入 `some_package`，因为它共享主库的导入
- 外部代码只需要 `import 'package:my_lib/main.dart'`，就能访问 `MainClass` 和 `HelperClass`

## Sizer 的代码组织

### 主库文件：sizer.dart

让我们详细分析主库文件：

```dart 1:17:lib/sizer.dart
/*
 * Created by Prashant Padmani on 2018/9/29.
 * email: prashant09mca@gmail.com
*/
library sizer;

import 'dart:math';

import 'package:flutter/widgets.dart';
import 'package:flutter/foundation.dart' show defaultTargetPlatform, kIsWeb;

part '../sizer-source-code-tutorial/extension.dart';

part '../sizer-source-code-tutorial/util.dart';

part '../sizer-source-code-tutorial/widget.dart';
```

**关键点分析**：

1. **库声明**：`library sizer;` 定义库名为 `sizer`
2. **导入 `dart:math`**：用于 `min` 和 `max` 函数（在 extension.dart 中使用）
3. **导入 Flutter 核心包**：
   - `package:flutter/widgets.dart`：Widget 系统
   - `package:flutter/foundation.dart`：平台检测相关
4. **Part 声明**：按顺序声明三个 part "../sizer-source-code-tutorial"文件

### Part 文件结构

让我们看看 part "../sizer-source-code-tutorial"文件的开头：

```dart 1:1:lib/extension.dart
part of sizer;
```

```dart 1:1:lib/util.dart
part of sizer;
```

```dart 1:1:lib/widget.dart
part of sizer;
```

所有 part "../sizer-source-code-tutorial"文件都声明 `part of sizer`，表示它们属于 `sizer` 库。

## 为什么使用 part"../sizer-source-code-tutorial"？

### 优势

#### 1. 共享导入和作用域

所有 part "../sizer-source-code-tutorial"文件自动共享主库的导入，无需重复导入：

- `extension.dart` 可以直接使用 `dart:math` 的 `min` 和 `max`
- `util.dart` 可以直接使用 Flutter 的 Widget 类
- `widget.dart` 可以直接使用 `Device` 类（定义在 `util.dart` 中）

如果没有使用 `part"../sizer-source-code-tutorial"`，每个文件都需要：

```dart
// extension.dart（如果独立）
import 'package:sizer/util.dart'; // 需要导入
import 'dart:math'; // 需要重复导入

extension SizerExt on num {
  // ...
}
```

#### 2. 内部类和方法可见性

使用 `part"../sizer-source-code-tutorial"` 后，可以在文件之间访问私有成员：

```dart
// util.dart
part of sizer;

class _InternalHelper {  // 私有类
  // ...
}

// widget.dart
part of sizer;

class Sizer {
  void method() {
    _InternalHelper(); // 可以访问，因为是同一库
  }
}
```

如果 `widget.dart` 是独立文件，就无法访问 `_InternalHelper`。

#### 3. 逻辑上的完整性

对于 Sizer 这样的库：

- `Device` 类（util.dart）是内部实现细节
- `SizerExt` 扩展（extension.dart）依赖 `Device` 类
- `Sizer` Widget（widget.dart）使用 `Device` 类

它们本质上是同一个库的不同部分，使用 `part"../sizer-source-code-tutorial"` 更符合逻辑关系。

#### 4. 导出控制

使用 `part"../sizer-source-code-tutorial"` 后，只有一个入口文件（`sizer.dart`），外部只需要：

```dart
import 'package:sizer/sizer.dart';
```

就能访问所有公开的 API。如果使用独立文件，可能需要：

```dart
import 'package:sizer/sizer.dart';
import 'package:sizer/extension.dart';  // 可能还需要这个
```

### 劣势

#### 1. 文件间耦合度高

所有 part "../sizer-source-code-tutorial"文件紧密耦合，修改一个文件可能影响其他文件。但这对 Sizer 这样的小型库来说不是问题。

#### 2. 不利于单独测试

Part 文件不能单独导入和测试，必须测试整个库。但这可以通过测试整体功能来弥补。

#### 3. 文件大小管理

如果库很大，所有 part "../sizer-source-code-tutorial"文件合并后可能很大。但 Sizer 的代码量适中，不存在这个问题。

## 代码组织的最佳实践

### 何时使用 part

"../sizer-source-code-tutorial"适合使用 `part"../sizer-source-code-tutorial"` 的场景：

1. **逻辑相关的代码**：文件之间紧密相关，属于同一个概念单元
2. **共享大量导入**：多个文件需要相同的导入
3. **私有成员访问**：需要访问其他文件的私有成员
4. **库的大小适中**：代码量不会过大

### 何时不使用 part

"../sizer-source-code-tutorial"不适合使用 `part"../sizer-source-code-tutorial"` 的场景：

1. **大型库**：代码量很大，需要更好的模块化
2. **独立功能模块**：功能相对独立，可以单独使用
3. **需要单独导出**：某些文件需要单独导出给外部使用
4. **团队协作**：多人协作时，独立文件冲突更少

## Sizer 的导入分析

让我们看看主库文件的导入：

```dart 7:10:lib/sizer.dart
import 'dart:math';

import 'package:flutter/widgets.dart';
import 'package:flutter/foundation.dart' show defaultTargetPlatform, kIsWeb;
```

### dart:math

`dart:math` 提供了 `min` 和 `max` 函数，在 `extension.dart` 中用于计算 `vmin` 和 `vmax`：

```dart 39:42:lib/extension.dart
  double get vmin => this * min(Device.height, Device.width) / 100;

  /// Respective percentage of the viewport's larger dimension.
  double get vmax => this * max(Device.height, Device.width) / 100;
```

### package:flutter/widgets.dart

提供了 Flutter Widget 系统的核心类：

- `Widget`、`StatelessWidget`、`BuildContext` 等（widget.dart 使用）
- `BoxConstraints`、`Orientation`、`LayoutBuilder` 等（util.dart 和 widget.dart 使用）

### package:flutter/foundation.dart

使用 `show` 关键字只导入需要的符号：

- `defaultTargetPlatform`：用于判断平台类型
- `kIsWeb`：用于判断是否为 Web 平台

这种选择性导入可以减少命名冲突，提高代码可读性。

## 文件依赖关系

Sizer 的文件之间存在以下依赖关系：

```text
sizer.dart (主库)
├── util.dart
│   ├── Device 类
│   └── Adaptive 类
├── extension.dart
│   └── SizerExt 扩展（依赖 Device 类）
└── widget.dart
    └── Sizer Widget（依赖 Device 类）
```

**依赖分析**：

- `extension.dart` 依赖 `Device` 类（在 `util.dart` 中定义）
- `widget.dart` 依赖 `Device` 类（在 `util.dart` 中定义）
- `util.dart` 不依赖其他 part "../sizer-source-code-tutorial"文件

由于使用 `part"../sizer-source-code-tutorial"` 机制，这些依赖关系在编译时就已经确定，不需要显式的导入语句。

## 总结

Sizer 使用 `part"../sizer-source-code-tutorial"` 机制组织代码，这种方式：

- **简化了导入**：所有 part "../sizer-source-code-tutorial"文件共享主库的导入
- **保持了逻辑完整性**：相关代码组织在同一个库中
- **控制了可见性**：可以访问同一库的私有成员
- **提供了单一入口**：外部只需导入 `sizer.dart`

对于 Sizer 这样的小型库，使用 `part"../sizer-source-code-tutorial"` 是合适的选择。

## 检查清单

在进入下一章之前，确保你理解：

- [ ] Dart 的 `part"../sizer-source-code-tutorial"` 和 `part of"../sizer-source-code-tutorial"` 机制的工作原理
- [ ] Sizer 为什么要使用 `part"../sizer-source-code-tutorial"` 而非独立文件
- [ ] `part"../sizer-source-code-tutorial"` 机制的优势和劣势
- [ ] Sizer 的文件依赖关系
- [ ] 何时应该使用 `part"../sizer-source-code-tutorial"`，何时不应该

## 实践练习

1. 尝试将 Sizer 的代码改为使用独立文件的方式，对比两种方式的差异。

2. 思考一下，如果 Sizer 库继续扩展，添加更多功能模块，是否还应该使用 `part"../sizer-source-code-tutorial"`？为什么？

3. 查看其他 Flutter 库的代码组织方式，看看它们是否使用 `part"../sizer-source-code-tutorial"`，分析原因。
