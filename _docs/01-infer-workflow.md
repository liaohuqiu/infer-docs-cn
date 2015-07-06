---
id: infer-workflow
title: Infer 的工作流程
layout: docs
permalink: /docs/infer-workflow.html
section: User Guide
section_order: 01
order: 01
---

本页文档说明 Infer 的几种运行方式，你可根据你自己的项目具体情况选用。

**摘要**

1. 初次运行时，确保项目是清理过的。可以通过 (`make clean`，`gradle clean` 等等)
2. 两次运行之间，记得清理项目，或者通过 `--incremental` 选项方式，因为增量编译而无结果输出。
3. 如果你使用的是非增量编译系统，则无需如此，比如：`infer -- javac Hello.java`，编译 Java 文件。
4. 成功运行之后，在同一目录下，你可以通过 `inferTraceBugs` 浏览更加详细的报告。

## Infer 运行的两个阶段

不管是哪种语言，Infer 运行时，分为两个主要阶段：

### 1. 捕获阶段

Infer 捕获编译命令，将文件翻译成 Infer 内部的中间语言。

这种翻译和编译类似，Infer 从编译过程获取信息，并进行翻译。这就是我们调用 Infer 时带上一个编译命令的原因了，比如: `infer -- clang -c file.c`, `infer -- javac File.java`。结果就是文件照常编译，同时被 Infer
翻译成中间语言，留作第二阶段处理。特别注意的就是，如果没有文件被编译，那么也没有任何文件会被分析。

Infer 把中间文件存储在结果文件夹中，一般来说，这个文件夹会在运行 `infer` 的目录下创建，命名是 `infer-out/`。当然，你也可以通过 `-o` 选项来自定义文件夹名字：

```bash
infer -o /tmp/out -- javac Test.java
```

### 2. 分析阶段

在这个阶段，Infer 会分析 `infer-out/` 目录下的文件。代码中的每个方法和函数都会被独立分析。如果在分析过程中遇到了问题，Infer 会停止当前的分析并停留在出现问题的地方，但是还会继续分析其他方法和函数。因此一个比较可行的方式就是就首先运行 Infer，修正出现的错误后再次运行来发现更多错误或者检查一下是否所有错误已经被修正。

错误会同时输出到标准输出设备和 `infer-out/bugs.txt` 文件中，我们进行过滤之后会输出最有可能的那些bug。同时，在结果目录（`infer-out/`） 下我们也会把所有的错误、警告以及其他信息保存在 `report.csv` 文件中。


## 增量和非增量方式的工作流程

默认情况下，执行 Infer 会删除前一次执行生成的 `infer-out` 目录，这是 *非增量* 的方式。 如果加上了参数 `--incremental` （或 `-i`），`infer-out` 目录则不会被删除，这是 *增量* 的方式。

当然也有例外情况，特别是只能执行某一个阶段时，例如，执行 `infer -- javac Hello.java` 就等价于以下两个命令。

```bash
infer -a capture -- javac Hello.java
infer -- analyze
```

需要注意的是，第二条命令并不会删除 `infer-out/` 文件夹，因为 Infer 要基于这些文件做后续分析。运行 `infer --help` 可以产看更多 Infer 的运行模式。

接下来我们会着重强调可能需要区分非增量和增量方式的情况。

### 非增量方式的工作流程

非增量方式的工作流程特别适用于只用一个编译命令来反复执行 Infer 的情况，例如：

```bash
infer -- javac Hello.java
edit Hello.java
# make some changes to Hello.java, e.g. fix a bug reported by Infer
infer -- javac Hello.java
```

如果要重新进行分析，必须完成以下动作：

1. 删除结果目录

    ```bash
    rm -fr infer-out
    ```

2. 清除 build 产物, 例如基于 make 的工程可以执行 `make clean`。


### 增量方式的工作流程

像移动应用开发这样的项目，需要使用**增量的**的构建系统，[下一节](docs/analyzing-apps-or-projects.html) 详细介绍了 Infer 如何利用这类系统。

Infer 必须要基于这类构建系统才能够进行项目分析。执行 Infer 非常简单 - `infer -- <构建命令>` ，其中构建命令就是平时用于编译代码的命令。为了在捕获阶段捕获所有的编译命令，需要在一个**干净**的项目上执行 Infer。

例如，一个利用 Gradle 的项目：

```bash
gradle clean
infer -- gradle build
```

接下来，如果修改了项目文件（例如修复 Infer 报告的bug），你可以重复上面的命令（因为项目被 clean 过，所以会重新分析）或者告诉 Infer 你要使用增量方式的工具链。


```bash
edit some/File.java
# make some changes to some/File.java
infer --incremental -- gradle build
```

注意：第一次执行 Infer 时也可以使用 `--incremental` 参数。

## 查看 Infer 报告

运行 `inferTraceBugs` 可以获取更多报告信息。

```bash
infer -- gradle build
inferTraceBugs
```
利用这个工具可以追踪到导致bug的错误信息，并且有助于精确定位错误原因。通过 `inferTraceBugs --help` 可获取更多帮助信息。
