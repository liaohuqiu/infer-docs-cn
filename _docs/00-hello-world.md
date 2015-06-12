---
id: hello-world
title: Hello, World!
layout: docs
permalink: /docs/hello-world.html
section: Quick Start
section_order: 00
order: 02
---

根据本页的说明，你可以使用 Infer 尝试检查一些简单的例子。你会看到 Infer 报告了一些问题，修复这些问题，再进行一次检测，Infer 便不再报告问题。这将使得我们对 Infer 如何工作有一些初步的认识，更近一步的使用，可以参考[用户使用参考](docs/infer-workflow.html)。


以下所有的例子都可以在 [`infer/examples`](https://github.com/facebook/infer/tree/master/examples) 中找到。

- [Hello world Java](docs/hello-world.html#hello-world-java)
- [Hello world Objective-C](docs/hello-world.html#hello-world-objective-c)
- [Hello world C](docs/hello-world.html#hello-world-c)
- [Hello world Android](docs/hello-world.html#hello-world-android)
- [Hello world iOS](docs/hello-world.html#hello-world-ios)
- [Hello world Make](docs/hello-world.html#hello-world-make)

## Hello world Java

以下是一个简答的 Java 例子。

```java
// Hello.java
class Hello {
  int test() {
    String s = null;
    return s.length();
  }
}
```

通过以下命令在 [`Hello.java`](https://github.com/facebook/infer/tree/master/examples/Hello.java) 同级目录执行 Infer。

```bash
infer -- javac Hello.java
```

你将会看到以下的报告输出：

```bash
Hello.java:5: error: NULL_DEREFERENCE
  object s last assigned on line 4 could be null and is dereferenced at line 5  
```

编辑文件，加入为空检查：

```java
  int test() {
    String s = null;
    return s == null ? 0 : s.length();
  }
```

再次运行 Infer，这次，运行结果显示 `No issues found`，未发现错误。

## Hello world Objective-C

以下是一个简单的 Objective-C 例子：

```Objective-C
// Hello.m
#import <Foundation/Foundation.h>

@interface Hello: NSObject
@property NSString* s;
@end

@implementation Hello
NSString* m() {
    Hello* hello = nil;
    return hello->_s;
}
@end
```

在 [`Hello.m`](https://github.com/facebook/infer/tree/master/examples/Hello.m) 同级目录，运行：

```bash
infer -- clang -c Hello.m
```

以下是错误报告输出：

```
Hello.m:10 NULL_DEREFERENCE
  pointer hello last assigned on line 9 could be null and is dereferenced at line 10, column 12
```

编辑，如下修正：

```Objective-C
NSString* m() {
    Hello* hello = nil;
    return hello.s;
}
```

再次运行，```No issues found```， 没有报错。

## Hello world C

一个简单的 C 例子：

```c
// hello.c
#include <stdlib.h>

void test() {
  int *s = NULL;
  *s = 42;
}
```

在 [`hello.c`](https://github.com/facebook/infer/tree/master/examples/hello.c) 同级目录运行：

```bash
infer -- gcc -c hello.c
```

报错输出：

```
hello.c:5: error: NULL_DEREFERENCE
  pointer s last assigned on line 4 could be null and is dereferenced at line 5, column 10
```

编辑，修正：

```c
void test() {
  int *s = NULL;
  if (s != NULL) {
    *s = 42;
  }
}
```

再次运行，不在汇报错误，问题修复。

在进行 C 文件分析时，Infer 使用 gcc 命令并在内部运行 clang 来解析。

因此，你可能会得到一些和 gcc 不一样的编译器警告以及错误。以下命令等效的：

```bash
infer -- gcc -c hello.c
infer -- clang -c hello.c
```

## Hello world Android

为了能够分析 Android 应用，请确保已经安装最新的 [Android
SDK](https://developer.android.com/sdk/installing/index.html) 22，并保持最新。当然还有 `Android SDK Build-tools` 和 `Android Support Repository`。

Android 的示例在 [`infer/examples/android_hello`](https://github.com/facebook/infer/tree/master/examples/android_hello/) 。
编辑 `local.properties` 并设定本地 SDK 路径。

Android 示例使用 [gradle](https://gradle.org/) 构建。不过你不需要下载和安装 gradle。
项目中的脚本 [`gradlew`](https://docs.gradle.org/current/userguide/gradle_wrapper.html) 会自动下载 gradle以及相关的项目依赖。

之后，运行：

```bash
infer -- ./gradlew build
```

Infer 会输出一系列问题：

```bash
MainActivity.java:20: error: NULL_DEREFERENCE
   object s last assigned on line 19 could be null and is dereferenced at line 20

MainActivity.java:37: error: RESOURCE_LEAK
   resource acquired by call to FileOutputStream(...) at line 34 is not released after line 37
```

### 增量分析

如果你没有改变任何文件，运行 Infer 进行检测，你会注意到，这次并没有进行分析。

这时因为 gradle 是增量的编译的。所有已经编译过的文件没有变动的文件将不会再编译。Infer 根据编译命令获取需要分析的文件，因此，这样的情况下，获取不到任何需要编译分析和分析的文件。

以下有两种解决方案：

1. 在两次分析之间运行 `gradlew clean`

    ```bash
   ./gradlew clean
   ```
   
   这将会使得每次都重新进行编译，所以 Infer 能获取到所有要分析的文件。

2. 在增量模式下运行 Infer

    ```bash
    infer --incremental -- ./gradlew build
    ```

    这会使得 Infer 保留之前的编译结果。如果没有 `--incremental` (短命令是 `-i` )，Infer 会移除存储这些结果的文件夹： `infer-out`。

你可以在 [Infer workflow](docs/infer-workflow.html) 页面中，详细了解这两种方案的细节。


## Hello world iOS

iOS 的示例代码在这里 [`infer/examples/ios_hello`](https://github.com/facebook/infer/tree/master/examples/ios_hello/)，使用 Infer 检测：

```bash
infer -- xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```

将会有以下问题输出：

```bash
AppDelegate.m:20: error: MEMORY_LEAK
   memory dynamically allocated to shadowPath by call to CGPathCreateWithRect() at line 20, column 28 is not reachable after line 20, column 5

AppDelegate.m:25: error: RESOURCE_LEAK
   resource acquired to fp by call to fopen() at line 25, column 8 is not released after line 25, column 5

AppDelegate.m:29: warning: PARAMETER_NOT_NULL_CHECKED
   Parameter callback is not checked for null, there could be a null pointer dereference: pointer callback could be null and is dereferenced at line 29, column 5

AppDelegate.m:34: error: NULL_DEREFERENCE
   pointer str last assigned on line 33 could be null and is dereferenced at line 34, column 12

AppDelegate.m:39: error: PREMATURE_NIL_TERMINATION_ARGUMENT
   pointer str last assigned on line 38 could be nil which results in a call to arrayWithObjects: with 1 arguments instead of 3 (nil indicates that the last argument of this variadic method has been reached) at line 39, column 12

Hello.m:20: error: NULL_DEREFERENCE
   pointer hello last assigned on line 19 could be null and is dereferenced at line 20, column 12

Hello.m:25: warning: IVAR_NOT_NULL_CHECKED
   Instance variable hello -> _hello is not checked for null, there could be a null pointer dereference: pointer ret_hello last assigned on line 24 could be null and is dereferenced at line 25, column 12

Hello.m:30: warning: PARAMETER_NOT_NULL_CHECKED
   Parameter hello is not checked for null, there could be a null pointer dereference: pointer ret_hello last assigned on line 29 could be null and is dereferenced at line 30, column 12
```

和[gradle](docs/hello-world.html#incremental-analysis) 相似，需要使用 `--incremental` (或 `-i`) 使得增量编译有结果输出。

```bash
infer --incremental -- xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```
或者在编译前清理：

```bash
xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator clean
```


## Hello world Make

使用 Infer 检测 C 代码，示例在 [`infer/examples/c_hello`](https://github.com/facebook/infer/tree/master/examples/c_hello/)。

```bash
infer -- make
```
将会输出以下问题：

```bash
example.c:22: error: NULL_DEREFERENCE
   pointer max last assigned on line 21 could be null and is dereferenced at line 22, column 10

example.c:36: error: NULL_DEREFERENCE
   pointer joe last assigned on line 35 could be null and is dereferenced by call to get_age() at line 36, column 10

example.c:45: error: RESOURCE_LEAK
   resource acquired to fd by call to open() at line 41, column 12 is not released after line 45, column 5

example.c:51: error: MEMORY_LEAK
   memory dynamically allocated to p by call to malloc() at line 51, column 14 is not reachable after line 51, column 3

example.c:57: error: MEMORY_LEAK
   memory dynamically allocated to p by call to malloc() at line 56, column 14 is not reachable after line 57, column 3
```

同样，和[gradle](docs/hello-world.html#incremental-analysis) 类似，为了使得增量编译下有结果输出，需要使用 `--incremental` (或 `-i`) 

```bash
infer --incremental -- make
```

或者清理：

```bash
make clean
```
