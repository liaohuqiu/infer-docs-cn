---
id: infer-workflow
title: Infer 的工作机制
layout: docs
permalink: /docs/infer-workflow.html
section: User Guide
section_order: 01
order: 01
---

本页文档说明 Infer 的几种运行方式，你可根据你自己的项目具体情况选用。

**摘要**

1. 初次运行时，确保项目是清理过的。可以通过 (`make clean`，`gradle clean`
等等)
2. 两次运行之间，记得清理项目，或者通过 `--incremental`
选项，方式因为增量编译而无结果输出。
3. 如果你使用的是非增量编译系统，则无需如此，比如：`infer -- javac Hello.java`，编译 Java 文件。
4. 成功运行之后，在同一目录下，你可以通过 `inferTraceBugs` 浏览更加详细的报告。

## Infer 运行的两个阶段

不管是分析哪种语言，Infer 运行时，分为两个主要阶段：

### 1. 捕获阶段

Infer 捕获编译命令，将文件翻译成 Infer 内部的中间语言。

这种翻译和编译类似，Infer 从编译过程获取信息，并进行翻译。这就是我们调用 Infer
时带上一个编译命令的原因了，比如: `infer -- clang -c file.c`, `infer -- javac
File.java`。结果就是文件照常编译，同时被 Infer
翻译成中间语言，留作第二阶段处理。特别注意的就是，如果没有文件被编译，那么也没有任何文件会被分析。

Infer 把中间文件存储在结果文件夹中，一般来说，这个文件夹会在运行 `infer`
的目录下创建，命名是 `infer-out/`。当然，你也可以通过 `-o`
选项来自定义文件夹名字：

```bash
infer -o /tmp/out -- javac Test.java
```

### 2. 分析阶段

在分析阶段，Infer 分析 `infer-out/` 下的所有文件。分析时，会单独分析每个方法和函数。

在分析一个函数的时候，如果发现错误，将会停止分析，但这不影响其他函数的继续分析。

所以你在检查问题的时候，修复输出的错误之后，需要继续运行 Infer 进行检查，知道确认所有问题都已经修复。

错误除了会显示在标准输出之外，还会输出到文件 `infer-out/bug.txt` 中，我们过滤这些问题，仅显示最有可能存在的。

在结果文件夹中（`infer-out`），同时还有一个 csv 文件 `report.csv`，这里包含了所有 Infer 产生的信息，包括：错误，警告和信息。

## 增量模式和非增量模式

运行时，Infer 默认会删除之前产生的 `infer-out/` 文件夹，这会导致非增量模式。

如果需要增量模式，加入 `--incremental`（或者 `-i`）参数运行，这样 `infer-out/` 文件夹将不会被删除。

也有例外的情况，尤其是你只能在上面的一个阶段使用这个参数。

比如， `infer -- javac Hello.java` 相当于运行了以下两个命令。

```bash
infer -a capture -- javac Hello.java
infer -- analyze
```

注意，第二个命令不会删除 `infer-out/`，因为分析阶段，需要使用文件夹中的进行分析。

你可以通过 `infer --help` 了解更多关于 Infer 的各种操作模式。

下面我们简单明了地强调一下，什么情况下需要使用增量模式，什么时候使用非增量模式。

### 非增量模式

非增量模式适用于单编译命令，重复运行 Infer 检测的情况。

```bash
infer -- javac Hello.java
edit Hello.java
# 编译 Hello.java，改动了代码，比如修复了一些问题
infer -- javac Hello.java
```

如果需要进行全新的一轮的分析，必须：

1. 删除结果文件夹：

    ```bash
    rm -fr infer-out
    ```

2. 删除构建产物。比如，对于基于 make 的项目，运行 `make clean`。


### 增量模式

许多软件项目都使用增量编译系统，比如手机应用。Infer 支持好些这样的编译系统，具体的看[这个章节](docs/analyzing-apps-or-projects.html). 

如果想使用 Infer 进行增量分析，你的编译系统需要是这其中的一个。

运行 Infer 进行检测的时候，只需要简单运行 `infer -- <编译命令>`，其中编译命令就是我们平时编译的命令。需要注意的是，运行前的项目是清理过的，这样 Infer 才能在捕获阶段捕获所有的编译命令。

比如，一个 gradle 项目：

```bash
gradle clean
infer -- gradle build
```

接下来，如果你修改了项目中的一些文件，你可以重复上面的命令，清理并重新分析整个项目。或者通过参数，让 Infer 使用使用增量模式。


```bash
edit some/File.java
# 修改了 some/File.java 的一些内容。
infer --incremental -- gradle build
```

当然，你也可以在第一次运行 Infer 进行检测的时候，就使用 `--incremental`。

## 查看详细报告信息

在同一目录，通过命令 `inferTraceBugs`，你可以查看报告中的更多信息：

```bash
infer -- gradle build
inferTraceBugs
```

通过这个工具，你可以导致 bug 的错误堆栈，有助于追踪问题的详细原因。具体的，使用 `inferTraceBugs --help` 查看使用帮助。
