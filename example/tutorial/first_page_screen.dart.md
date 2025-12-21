# FirstPageScreen 代码讲解

## 概述

`first_page_screen.dart` 是一个展示如何使用 Sizer 库创建响应式 Flutter 页面的示例。该页面实现了横竖屏自适应布局，通过 Sizer 提供的扩展方法，使 UI 能够自动适配不同屏幕尺寸的设备。

### 核心特性

- **响应式布局**：根据屏幕方向（横屏/竖屏）自动切换布局
- **自适应尺寸**：使用 Sizer 扩展方法实现基于屏幕百分比的尺寸计算
- **可缩放字体**：使用 `.sp` 单位实现字体大小的自适应

## 代码结构

### 导入依赖

```dart 1:5:example/lib/screens/first_page_screen.dart
import 'package:example/util/constant.dart';
import 'package:example/util/strings.dart';
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'package:sizer/sizer.dart';
```

代码导入了以下依赖：

- `constant.dart`：包含图片路径等常量定义
- `strings.dart`：包含应用文本字符串
- `flutter/material.dart`：Flutter 基础 Material 组件
- `google_fonts`：Google 字体库，用于使用 Lato 字体
- `sizer`：Sizer 库，提供响应式尺寸扩展方法

### 类定义

```dart 7:10:example/lib/screens/first_page_screen.dart
class FirstPageScreen extends StatefulWidget {
  @override
  _FirstPageScreenState createState() => _FirstPageScreenState();
}
```

`FirstPageScreen` 继承自 `StatefulWidget`，用于创建有状态的页面组件。由于需要根据屏幕方向动态切换布局，使用 `StatefulWidget` 是合适的选择。

### 状态类

```dart 12:18:example/lib/screens/first_page_screen.dart
class _FirstPageScreenState extends State<FirstPageScreen> {
  @override
  Widget build(BuildContext context) {
    return Device.orientation == Orientation.portrait
        ? _widPortrait()
        : _widLandScape();
  }
```

`_FirstPageScreenState` 是状态类，其 `build` 方法的核心逻辑是：

1. 使用 `Device.orientation` 检测当前屏幕方向
2. 如果是竖屏（`Orientation.portrait`），调用 `_widPortrait()` 方法
3. 如果是横屏（`Orientation.landscape`），调用 `_widLandScape()` 方法

## 核心功能详解

### 竖屏布局实现

```dart 20:38:example/lib/screens/first_page_screen.dart
  Widget _widPortrait() {
    return Material(
      child: SingleChildScrollView(
        child: Container(
          height: 100.h,
          child: Column(
            mainAxisSize: MainAxisSize.max,
            children: [
              _widMainImg(),
              Expanded(
                  child: Column(
                children: [_widTitle(), _widDesc()],
              )),
            ],
          ),
        ),
      ),
    );
  }
```

竖屏布局的结构层次：

1. **Material**：提供 Material Design 样式的基础容器
2. **SingleChildScrollView**：使内容可滚动，防止内容溢出
3. **Container**：使用 `100.h` 占满屏幕高度（100% 屏幕高度）
4. **Column**：垂直排列子组件
   - `_widMainImg()`：主图片组件
   - `Expanded`：占据剩余空间
     - 内部 `Column` 包含标题和描述文本

#### 主图片组件

```dart 40:45:example/lib/screens/first_page_screen.dart
  _widMainImg() {
    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 3.w),
      child: Image.asset(Constant.IMG_1, height: 50.h),
    );
  }
```

- `padding: EdgeInsets.symmetric(horizontal: 3.w)`：左右各 3% 屏幕宽度的内边距
- `height: 50.h`：图片高度为屏幕高度的 50%

#### 标题组件

```dart 47:55:example/lib/screens/first_page_screen.dart
  _widTitle() {
    return Padding(
      padding: EdgeInsets.only(top: 1.5.h),
      child: Text('Sizer',
          style: GoogleFonts.lato(
            textStyle: TextStyle(fontSize: 30.sp, color: Colors.black),
          )),
    );
  }
```

- `padding: EdgeInsets.only(top: 1.5.h)`：顶部间距为屏幕高度的 1.5%
- `fontSize: 30.sp`：使用 30 个可缩放像素单位，会根据设备自动调整

#### 描述文本组件

```dart 57:68:example/lib/screens/first_page_screen.dart
  _widDesc() {
    return Padding(
        padding: EdgeInsets.only(right: 10.w, left: 10.w, top: 2.h),
        child: Text(
          Strings.APP_DESC,
          textAlign: TextAlign.center,
          style: GoogleFonts.lato(
            textStyle:
                TextStyle(height: 1.3, fontSize: 13.sp, color: Colors.grey),
          ),
        ));
  }
```

- `padding: EdgeInsets.only(right: 10.w, left: 10.w, top: 2.h)`：左右各 10% 屏幕宽度，顶部 2% 屏幕高度
- `fontSize: 13.sp`：使用 13 个可缩放像素单位
- `textAlign: TextAlign.center`：文本居中对齐

### 横屏布局实现

```dart 70:88:example/lib/screens/first_page_screen.dart
  _widLandScape() {
    return Material(
      child: SingleChildScrollView(
        child: Container(
          height: 100.h,
          child: Column(
            children: [
              _widMainImgLand(),
              Expanded(
                  child: Column(
                children: [_widTitleLand(), _widDescLand()],
              )),
            ],
          ),
        ),
      ),
    );
  }
```

横屏布局的整体结构与竖屏相同，但使用了不同的组件方法，这些方法针对横屏进行了参数调整。

#### 横屏主图片组件

```dart 90:95:example/lib/screens/first_page_screen.dart
  _widMainImgLand() {
    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 3.w),
      child: Image.asset(Constant.IMG_1, height: 25.h),
    );
  }
```

**关键差异**：图片高度从 `50.h` 调整为 `25.h`，因为横屏时屏幕高度较小，需要减少图片占用空间。

#### 横屏标题组件

```dart 97:105:example/lib/screens/first_page_screen.dart
  _widTitleLand() {
    return Padding(
      padding: EdgeInsets.only(top: 1.0.h),
      child: Text(Strings.APP_NAME,
          style: GoogleFonts.lato(
            textStyle: TextStyle(fontSize: 30.sp, color: Colors.black),
          )),
    );
  }
```

**关键差异**：

- 顶部间距从 `1.5.h` 调整为 `1.0.h`
- 使用 `Strings.APP_NAME` 而不是硬编码的 'Sizer'

#### 横屏描述文本组件

```dart 107:118:example/lib/screens/first_page_screen.dart
  _widDescLand() {
    return Padding(
        padding: EdgeInsets.only(right: 25.w, left: 25.w, top: 1.5.h),
        child: Text(
          Strings.APP_DESC,
          textAlign: TextAlign.center,
          style: GoogleFonts.lato(
            textStyle:
                TextStyle(height: 1.3, fontSize: 13.sp, color: Colors.grey),
          ),
        ));
  }
```

**关键差异**：

- 左右内边距从 `10.w` 调整为 `25.w`，因为横屏时屏幕宽度较大，可以留出更多边距
- 顶部间距从 `2.h` 调整为 `1.5.h`

## Sizer 扩展方法详解

### `.h` - 高度百分比

`数字.h` 表示屏幕高度的百分比。例如：

- `100.h` = 屏幕高度的 100%
- `50.h` = 屏幕高度的 50%
- `1.5.h` = 屏幕高度的 1.5%

**实现原理**：`double get h => this * Device.height / 100`

### `.w` - 宽度百分比

`数字.w` 表示屏幕宽度的百分比。例如：

- `100.w` = 屏幕宽度的 100%
- `10.w` = 屏幕宽度的 10%
- `3.w` = 屏幕宽度的 3%

**实现原理**：`double get w => this * Device.width / 100`

### `.sp` - 可缩放像素

`数字.sp` 表示可缩放像素单位，会根据设备的像素密度和宽高比自动调整。例如：

- `30.sp` = 30 个可缩放像素单位
- `13.sp` = 13 个可缩放像素单位

**实现原理**：基于设备高度、宽度、像素密度和宽高比进行综合计算，确保在不同设备上字体大小保持相对一致的可读性。

## 与不使用 Sizer 的版本对比

为了理解使用 Sizer 的优势，我们可以对比 `first_page_screen_without_sizer.dart` 文件。

### 不使用 Sizer 的实现

在 `first_page_screen_without_sizer.dart` 中：

```dart
widMainImg() {
  return Padding(
    padding: EdgeInsets.symmetric(horizontal: 15.0),
    child: Image.asset(Constant.IMG_1, height: MediaQuery.of(context).size.height/2),
  );
}
```

- 使用固定像素值：`horizontal: 15.0`
- 使用 `MediaQuery.of(context).size.height/2` 计算高度

```dart
widTitle() {
  return Text(Strings.APP_NAME,
      style: GoogleFonts.lato(
        textStyle: TextStyle(fontSize: 35.0, color: Colors.black),
      ));
}
```

- 使用固定字体大小：`fontSize: 35.0`

### 使用 Sizer 的优势

1. **代码简洁**：`3.w` 比 `MediaQuery.of(context).size.width * 0.03` 更简洁易读
2. **语义清晰**：`50.h` 直接表达"屏幕高度的 50%"，意图明确
3. **统一管理**：所有响应式尺寸使用统一的扩展方法，便于维护
4. **自动适配**：`.sp` 单位自动处理不同设备的像素密度差异
5. **减少重复**：不需要在每个组件中调用 `MediaQuery.of(context)`

## 最佳实践建议

### 何时使用 `.h`、`.w`、`.sp`

- **`.h`**：适用于需要基于屏幕高度的尺寸，如容器高度、垂直间距、图片高度等
- **`.w`**：适用于需要基于屏幕宽度的尺寸，如容器宽度、水平间距、左右内边距等
- **`.sp`**：适用于字体大小，确保在不同设备上保持良好的可读性

### 横竖屏适配注意事项

1. **分别实现布局方法**：为横屏和竖屏分别创建布局方法，可以更精细地控制不同方向下的 UI
2. **调整关键参数**：
   - 横屏时屏幕高度较小，应减少垂直方向的空间占用
   - 横屏时屏幕宽度较大，可以增加水平方向的内边距
3. **保持一致性**：横竖屏布局应保持相同的视觉风格和交互逻辑

### 性能考虑

- Sizer 的扩展方法计算开销很小，可以放心使用
- `Device.orientation` 的检测是高效的，不会造成性能问题
- 使用 `SingleChildScrollView` 可以防止内容溢出，同时保持良好的滚动性能

## 总结

`first_page_screen.dart` 展示了如何使用 Sizer 库创建响应式 Flutter 页面。通过使用 `.h`、`.w`、`.sp` 等扩展方法，代码变得简洁易读，同时实现了良好的跨设备适配效果。横竖屏分别实现布局方法，使得 UI 在不同屏幕方向下都能提供良好的用户体验。
