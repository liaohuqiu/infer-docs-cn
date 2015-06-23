---
id: advanced-features
title: 高级用法
layout: docs
permalink: /docs/advanced-features.html
section: User Guide
section_order: 01
order: 06
---

如果你想详细了解 Infer 具体是如何工作的，或者想为 Infer 添砖加瓦，这个章节我们会具体讨论诸如调试选项，获取方法详细定义的办法等细节问题。

## 结果文件夹结构

检测成功运行之后，分析结果会被放入一个默认文件夹，`infer-out`：


```
infer-out
├── captured/
├── log/
├── multicore/
├── sources/
├── specs/
├── bugs.txt
├── procs.csv
├── report.csv
├── report.json
└── stats.json
```


- `captured/` 包含了 Infer 分析需要的每个文件的信息，具体看 [下面](docs/advanced-features.html#captured-folder)
- `log/`, `multicore/`, 和 `sources/` 文件夹是分析器内部驱动所需。
- `specs/` 包含了所分析的各个方法的 [参数指标](docs/advanced-features.html#print-the-specs)，Infer 据此推断文件。
- `bugs.txt`, `report.csv`, 和 `report.json` 为三种不同格式的分析结果。
- `procs.csv` and `stats.json` 包含 debug 信息的分析结果。


### 捕获文件夹

每个被捕获进行分析的文件在 `infer-out/captured` 下都有一个对应的文件夹。比如有一个文件名为 `example.c`，那么便会有一个文件夹 `infer-out/captured/example.c/`：

- `example.c.cfg`
- `example.c.cg`
- `example.c.tenv`

`.cfg`， `.cg` 和 `.tenv` 后缀的文件包含了所分析文件的中间表示，这些数据传送给 Infer 的后端程序进行分析。 这些文件包含了序列化后的 OCaml 数据结构。`.cfg` 文件包含了代码文件中每个函数或方法的控制流程。`.cg` 包含了代码文件中定义的函数的调用关系，以及该文件对外部函数的调用关系。 `.tenv` 包含了代码文件中定义和用到的类型。


## Debug 模式

通过 debug 选项 `infer --debug -- <build command>` 可输出调试信息。当使用 `make` 和 `clang` 的时候，需要加一个额外的前端调试选项：

```bash
infer --frontend_debug --debug -- make example.c
```

在每个捕获文件夹中，会有一个 `icfg.dot` 文件，这个文件是 `.cfg` 和 `call_graph.dot` 文件的图表示，也就是说，这是调用关系的图表示。

另外，在 `infer-out/captured` 文件夹下，每个文件还有一个 html 页面文件。这个文件包含了源代码文件，文件的每一行，都有链接指向该行代码对应的控制流程的节点。点击各个节点的链接可以查看各个节点的对应的符号执行的详细情况。如果开启了 `--no_test` 选项，节点详细情况页面将会包含打印出来的所有符号执行的情况。

## 输出方法的详细定义

使用 `InferPrint` 可以输出 Infer 创建的方法的详细定义，你可以输出一个方法或者所有方法的详细定义，如下：

```java
class Hello {
    int x;
    void setX(int newX) {
	    this.x = newX;
    }
}
```

运行：

```bash
	infer -- javac Hello.java
```

`setX` 的详细定义储存在 `infer-out/specs`，我们可以如下打印输出：

```bash
	InferPrint infer-out/specs/Hello.setX{98B5}:void.specs
```

Java 方法的命名规则为，`<class name>.<method name>`，输出如下：

```bash
Procedure: void Hello.setX(int)
void void Hello.setX(int)(class Hello *this, int newX)
Timestamp: 1
Status: INACTIVE
Phase: RE_EXECUTION
Dependency_map:
TIME:0.006893 s TIMEOUT:N SYMOPS:34 CALLS:0,0
ERRORS:
--------------------------- 1 of 1 [nvisited: 4 5 6] ---------------------------
PRE:
this = val$1: ;
newX = val$3: ;
this|->{Hello.x:val$2}:
POST 1 of 1:
this = val$1: ;
return = val$4: ;
newX = val$3: ;
this|->{Hello.x:newX}:
----------------------------------------------------------------
```

以上可见，方法执行最开始，`this` 需要初始化，在方法最后，`x` 和 `newX` 是相等的。

如果要打印结果文件夹下所有的方法的详细定义：

```bash
InferPrint -results_dir infer-out
```

## 运行内部测试用例

在 Infer 的代码中，有很多测试用例。我们使用这些测试用例来检验 Infer 行为的正确性。

运行这些测试用例时候，我们使用 Facebook 的另外一个开源项目 [Buck](http://buckbuild.com/) 来构建项目，我们有一个 `inferTest` 脚本来运行这些测试用例。使用这个脚本需要确保 buck 路径在 PATH 变量中。


```bash
inferTest java    # Run the tests about Java analysis
inferTest clang   # Run the tests about C and Objective-C analysis
inferTest c       # Run the tests about C analysis
inferTest objc    # Run the tests about Objective-C analysis
```
