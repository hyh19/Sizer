---
description: "Guide for using Sizer library for responsive Flutter UI design"
alwaysApply: false
---

# Sizer Library Usage Guide

## Overview

Sizer is a Flutter package that simplifies responsive UI design across different screen sizes and orientations. This guide provides comprehensive instructions for using Sizer effectively in Flutter applications.

## When to Use Sizer

Use Sizer when you need to:

- Create responsive layouts that adapt to different screen sizes
- Support both portrait and landscape orientations
- Distinguish between mobile, tablet, and desktop layouts
- Use percentage-based sizing instead of fixed pixel values
- Simplify MediaQuery usage with cleaner syntax

## Core Concepts

Sizer provides three main components:

1. **Sizer Widget**: Wraps your app and initializes device information
2. **Extension Methods**: Add responsive sizing methods to numbers (`.h`, `.w`, `.sp`, etc.)
3. **Device Class**: Provides access to device information (orientation, screen type, dimensions)

## Basic Setup

### Wrap MaterialApp with Sizer

Always wrap your `MaterialApp` with the `Sizer` widget in your `main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:sizer/sizer.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Sizer(
      builder: (context, orientation, screenType) {
        return MaterialApp(
          title: 'My App',
          home: HomePage(),
        );
      },
    );
  }
}
```

### Sizer Widget Parameters

The `Sizer` widget accepts optional parameters:

```dart
Sizer(
  builder: (context, orientation, screenType) {
    return MaterialApp(/* ... */);
  },
  maxMobileWidth: 599,  // Default: 599. Breakpoint between mobile and tablet
  maxTabletWidth: 1024, // Optional: Breakpoint between tablet and desktop
)
```

**Parameters:**

- `builder` (required): A function that receives `BuildContext`, `Orientation`, and `ScreenType`
- `maxMobileWidth` (optional, default: 599): Maximum width in portrait mode to consider a device as mobile
- `maxTabletWidth` (optional): Maximum width in portrait mode to consider a device as tablet. If set, enables `ScreenType.desktop`

## Extension Methods

Sizer provides extension methods on `num` types (int, double) for responsive sizing.

### Height and Width Percentages

Use `.h` and `.w` to get percentages of screen height and width:

```dart
Container(
  width: 50.w,   // 50% of screen width
  height: 30.h,  // 30% of screen height
)
```

**Best Practice**: Use `.h` and `.w` by default. They are based on the full screen dimensions.

### Safe Area Percentages

Use `.sh` and `.sw` for sizes based on safe area (remaining space after system UI):

```dart
Container(
  width: 50.sw,   // 50% of safe area width
  height: 30.sh,  // 30% of safe area height
)
```

**Note**: Only use `.sh` and `.sw` when you specifically need safe area calculations. Use `.h` and `.w` by default.

### Scalable Pixels

Use `.sp` for scalable pixel units that adjust based on device pixel density and aspect ratio:

```dart
Text(
  'Hello',
  style: TextStyle(fontSize: 16.sp),  // Responsive font size
)
```

### Density-Independent Pixels

Use `.dp` for Material Design density-independent pixels:

```dart
Container(
  width: 100.dp,  // Material Design dp units
  height: 50.dp,
)
```

### Viewport Units

Use `.vmin` and `.vmax` for viewport-relative units:

```dart
Container(
  width: 50.vmin,   // 50% of the smaller dimension (width or height)
  height: 50.vmax,  // 50% of the larger dimension (width or height)
)
```

**Use Case**: `.vmin` is useful for creating square containers that scale proportionally.

### Absolute Units

Sizer also supports absolute unit conversions:

```dart
100.cm      // Centimeters
100.mm      // Millimeters
100.inches  // Inches
100.pt      // Points (1/72 inch)
100.pc      // Picas (1/6 inch)
100.px      // Pixels (default)
```

## Device Class

The `Device` class provides static properties to access device information.

### Screen Dimensions

```dart
Device.width      // Screen width in pixels
Device.height     // Screen height in pixels
Device.safeWidth  // Safe area width
Device.safeHeight // Safe area height
```

### Orientation

Check the current screen orientation:

```dart
if (Device.orientation == Orientation.portrait) {
  // Portrait layout
} else {
  // Landscape layout
}
```

### Screen Type

Distinguish between mobile, tablet, and desktop:

```dart
if (Device.screenType == ScreenType.mobile) {
  // Mobile layout
} else if (Device.screenType == ScreenType.tablet) {
  // Tablet layout
} else if (Device.screenType == ScreenType.desktop) {
  // Desktop layout (only available if maxTabletWidth is set)
}
```

### Device Type

Get the platform type:

```dart
Device.deviceType == DeviceType.android
Device.deviceType == DeviceType.ios
Device.deviceType == DeviceType.web
Device.deviceType == DeviceType.windows
Device.deviceType == DeviceType.mac
Device.deviceType == DeviceType.linux
```

### Other Properties

```dart
Device.aspectRatio   // Screen aspect ratio
Device.pixelRatio    // Device pixel ratio
Device.boxConstraints // BoxConstraints of the device
```

## Responsive Patterns

### Orientation-Based Layout

Adjust layouts based on orientation:

```dart
Widget build(BuildContext context) {
  return Device.orientation == Orientation.portrait
    ? Column(
        children: [
          Container(width: 100.w, height: 30.h),
          Container(width: 100.w, height: 70.h),
        ],
      )
    : Row(
        children: [
          Container(width: 50.w, height: 100.h),
          Container(width: 50.w, height: 100.h),
        ],
      );
}
```

### Screen Type-Based Layout

Provide different layouts for different device types:

```dart
Widget build(BuildContext context) {
  if (Device.screenType == ScreenType.tablet) {
    return TabletLayout();
  } else {
    return MobileLayout();
  }
}
```

### Combined Responsive Logic

Combine multiple conditions:

```dart
Widget build(BuildContext context) {
  final isTablet = Device.screenType == ScreenType.tablet;
  final isPortrait = Device.orientation == Orientation.portrait;
  
  return Container(
    padding: EdgeInsets.all(isTablet ? 4.w : 2.w),
    child: GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: isTablet ? (isPortrait ? 3 : 4) : 2,
        crossAxisSpacing: 2.w,
        mainAxisSpacing: 2.h,
      ),
      itemBuilder: (context, index) => ItemWidget(),
    ),
  );
}
```

## Adaptive Class

The `Adaptive` class provides static methods as an alternative to extension methods:

```dart
// Using extension methods (preferred)
Container(width: 50.w, height: 30.h)

// Using Adaptive class (alternative)
Container(
  width: Adaptive.w(50),
  height: Adaptive.h(30),
)
```

**When to use**: Use `Adaptive` class when you prefer method calls over extension methods, or when working with dynamic values:

```dart
double calculateWidth(double percentage) {
  return Adaptive.w(percentage);  // Can't use extension methods in functions easily
}
```

All extension methods have corresponding `Adaptive` class methods:

- `Adaptive.h()`, `Adaptive.w()`
- `Adaptive.sh()`, `Adaptive.sw()`
- `Adaptive.sp()`, `Adaptive.dp()`
- `Adaptive.vmin()`, `Adaptive.vmax()`
- `Adaptive.cm()`, `Adaptive.mm()`, `Adaptive.inches()`, etc.

## Best Practices

### 1. Always Wrap MaterialApp with Sizer

Never use Sizer extension methods before the `Sizer` widget initializes the device information:

```dart
// ❌ WRONG - Will throw LateInitializationError
void main() {
  final width = 50.w;  // Device not initialized yet
  runApp(MyApp());
}

// ✅ CORRECT
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Sizer(
      builder: (context, orientation, screenType) {
        // Now you can use extension methods
        return MaterialApp(/* ... */);
      },
    );
  }
}
```

### 2. Prefer `.h` and `.w` Over `.sh` and `.sw`

Use `.h` and `.w` by default unless you specifically need safe area calculations:

```dart
// ✅ CORRECT - Use by default
Container(width: 50.w, height: 30.h)

// ⚠️ Only use when safe area is important
Container(width: 50.sw, height: 30.sh)
```

### 3. Handle Orientation Changes

Be aware that `.w` and `.h` values change when orientation changes:

```dart
// Portrait: 50.w might be 200px, 50.h might be 400px
// Landscape: 50.w might be 400px, 50.h might be 200px
Container(width: 50.w, height: 50.h)  // Dimensions swap in landscape
```

If you need consistent sizing across orientations, use `.vmin`:

```dart
// Square that maintains aspect ratio
Container(width: 30.vmin, height: 30.vmin)
```

### 4. Use Appropriate Units

- **Layout dimensions**: Use `.h` and `.w` for containers, spacing, etc.
- **Font sizes**: Use `.sp` for responsive text
- **Material Design**: Use `.dp` when following Material guidelines
- **Fixed sizes**: Use `.px` or fixed numbers for elements that shouldn't scale

```dart
Container(
  width: 80.w,           // Layout width
  height: 20.h,          // Layout height
  padding: EdgeInsets.all(2.w),  // Responsive padding
  child: Text(
    'Title',
    style: TextStyle(fontSize: 18.sp),  // Responsive font
  ),
)
```

### 5. Combine with MediaQuery When Needed

You can still use `MediaQuery` for context-specific queries, but Sizer provides a cleaner API for most cases:

```dart
// Using Sizer (preferred for screen dimensions)
Container(width: 50.w, height: 30.h)

// Using MediaQuery (for other MediaQuery features)
final padding = MediaQuery.of(context).padding;
final isDarkMode = MediaQuery.of(context).platformBrightness == Brightness.dark;
```

## Common Pitfalls and Errors

### Pitfall 1: Using Extension Methods Before Initialization

**Error**: `LateInitializationError` when accessing `Device` properties before `Sizer` widget builds.

**Solution**: Always use extension methods inside widgets that are descendants of the `Sizer` widget.

```dart
// ❌ WRONG
class MyWidget extends StatelessWidget {
  final double width = 50.w;  // Called at class definition time
  
  @override
  Widget build(BuildContext context) {
    return Container(width: width);
  }
}

// ✅ CORRECT
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(width: 50.w);  // Called during build, after Sizer initializes
  }
}
```

### Pitfall 2: Orientation Changes Cause Layout Issues

**Problem**: Layouts break when switching between portrait and landscape because dimensions swap.

**Solution**: Design layouts to handle orientation changes, or use `.vmin` for consistent sizing:

```dart
// ✅ Handles orientation changes
Widget build(BuildContext context) {
  return Device.orientation == Orientation.portrait
    ? VerticalLayout()
    : HorizontalLayout();
}
```

### Pitfall 3: Nested LayoutBuilder Conflicts

**Problem**: Using `LayoutBuilder` inside `Sizer` may give different constraints than `Device` properties.

**Solution**: Be aware that nested `LayoutBuilder` constraints may differ from `Device` properties if there are padding or other constraints in between.

### Pitfall 4: Forgetting to Import Sizer

**Error**: Extension methods not available.

**Solution**: Always import the package:

```dart
import 'package:sizer/sizer.dart';
```

### Pitfall 5: Mixing Extension Methods with BuildContext-Dependent Code

**Problem**: Extension methods don't need `BuildContext`, but you might forget this and unnecessarily pass context.

**Solution**: Extension methods work anywhere after `Sizer` initialization, no context needed:

```dart
// ✅ CORRECT - No context needed
double calculateSize() {
  return 50.w;
}

// ❌ UNNECESSARY - Context not needed
double calculateSize(BuildContext context) {
  return 50.w;  // Context parameter not used
}
```

## Complete Examples

### Example 1: Basic Responsive Container

```dart
import 'package:flutter/material.dart';
import 'package:sizer/sizer.dart';

class ResponsiveContainer extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      width: 80.w,
      height: 40.h,
      padding: EdgeInsets.all(2.w),
      margin: EdgeInsets.symmetric(horizontal: 5.w, vertical: 2.h),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(2.w),
      ),
      child: Center(
        child: Text(
          'Responsive Container',
          style: TextStyle(fontSize: 16.sp, color: Colors.white),
        ),
      ),
    );
  }
}
```

### Example 2: Responsive Grid Layout

```dart
import 'package:flutter/material.dart';
import 'package:sizer/sizer.dart';

class ResponsiveGrid extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final crossAxisCount = Device.screenType == ScreenType.tablet ? 4 : 2;
    
    return GridView.builder(
      padding: EdgeInsets.all(2.w),
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: crossAxisCount,
        crossAxisSpacing: 2.w,
        mainAxisSpacing: 2.h,
        childAspectRatio: 1.2,
      ),
      itemCount: 20,
      itemBuilder: (context, index) {
        return Container(
          decoration: BoxDecoration(
            color: Colors.grey[300],
            borderRadius: BorderRadius.circular(1.w),
          ),
          child: Center(
            child: Text(
              'Item $index',
              style: TextStyle(fontSize: 14.sp),
            ),
          ),
        );
      },
    );
  }
}
```

### Example 3: Orientation-Aware Layout

```dart
import 'package:flutter/material.dart';
import 'package:sizer/sizer.dart';

class OrientationLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Device.orientation == Orientation.portrait
      ? Column(
          children: [
            Container(
              width: 100.w,
              height: 30.h,
              color: Colors.blue,
              child: Center(
                child: Text('Header', style: TextStyle(fontSize: 20.sp)),
              ),
            ),
            Expanded(
              child: Container(
                width: 100.w,
                color: Colors.green,
                child: Center(
                  child: Text('Content', style: TextStyle(fontSize: 18.sp)),
                ),
              ),
            ),
          ],
        )
      : Row(
          children: [
            Container(
              width: 30.w,
              height: 100.h,
              color: Colors.blue,
              child: Center(
                child: Text('Sidebar', style: TextStyle(fontSize: 18.sp)),
              ),
            ),
            Expanded(
              child: Container(
                width: 70.w,
                color: Colors.green,
                child: Center(
                  child: Text('Content', style: TextStyle(fontSize: 18.sp)),
                ),
              ),
            ),
          ],
        );
  }
}
```

### Example 4: Complete App Setup with Sizer

```dart
import 'package:flutter/material.dart';
import 'package:sizer/sizer.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Sizer(
      builder: (context, orientation, screenType) {
        return MaterialApp(
          title: 'Sizer Example',
          theme: ThemeData(
            primarySwatch: Colors.blue,
          ),
          home: HomePage(),
        );
      },
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Sizer Example'),
      ),
      body: SafeArea(
        child: Padding(
          padding: EdgeInsets.all(4.w),
          child: Column(
            children: [
              Text(
                'Screen Info',
                style: TextStyle(fontSize: 24.sp, fontWeight: FontWeight.bold),
              ),
              SizedBox(height: 4.h),
              _InfoRow(label: 'Width', value: '${Device.width.toStringAsFixed(0)} px'),
              _InfoRow(label: 'Height', value: '${Device.height.toStringAsFixed(0)} px'),
              _InfoRow(label: 'Orientation', value: Device.orientation.toString().split('.').last),
              _InfoRow(label: 'Screen Type', value: Device.screenType.toString().split('.').last),
              SizedBox(height: 4.h),
              Container(
                width: 80.w,
                height: 20.h,
                decoration: BoxDecoration(
                  color: Colors.blue,
                  borderRadius: BorderRadius.circular(2.w),
                ),
                child: Center(
                  child: Text(
                    'Responsive Container',
                    style: TextStyle(fontSize: 18.sp, color: Colors.white),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class _InfoRow extends StatelessWidget {
  final String label;
  final String value;

  const _InfoRow({required this.label, required this.value});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 1.h),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: TextStyle(fontSize: 16.sp)),
          Text(value, style: TextStyle(fontSize: 16.sp, fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
```

## Summary

When using Sizer:

1. **Always wrap MaterialApp** with the `Sizer` widget
2. **Use `.h` and `.w`** for responsive dimensions (preferred over `.sh` and `.sw`)
3. **Use `.sp`** for responsive font sizes
4. **Check `Device.orientation`** for orientation-based layouts
5. **Check `Device.screenType`** for device-type-based layouts
6. **Never use extension methods** before `Sizer` widget initializes
7. **Design for orientation changes** - dimensions swap when rotating
8. **Combine with Flutter's layout widgets** for complete responsive designs

Sizer simplifies responsive design in Flutter by providing intuitive extension methods and device information, making it easier to create layouts that work across all screen sizes and orientations.
