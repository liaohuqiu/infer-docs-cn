---
id: advanced-features
title: 高级用法
layout: docs
permalink: /docs/advanced-features.html
section: User Guide
section_order: 01
order: 06
---

如果你想详细了解 Infer 具体是如何工作的，或者想为 Infer 添砖加瓦，这个章节我们会具体讨论诸如调试选项，获取方法详细参数指标的办法等细节问题。

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


- `captured/` 包含了 Infer 分析需要的每个文件的信息，具体看 [下面](docs/advanced-features.html#captured-folder) for more information。
- `log/`, `multicore/`, 和 `sources/` 文件夹是分析器内部驱动所需。
- `specs/` 包含了所分析的各个方法的 [参数指标](docs/advanced-features.html#print-the-specs)，Infer 据此推断文件。
- `bugs.txt`, `report.csv`, 和 `report.json` 为三种不同格式的分析结果。
- `procs.csv` and `stats.json` 包含 debug 信息的分析结果。


### 捕获文件夹

每个被捕获进行分析的文件在 `infer-out/captured` 下都有一个对应的文件夹。比如有一个文件名为 `example.c`，那么便会有一个文件夹 `infer-out/captured/example.c/`：

- `example.c.cfg`
- `example.c.cg`
- `example.c.tenv`

`.cfg`， `.cg` 和 `.tenv` 后缀的文件包含了所分析文件的中间表示，这些数据传送给 Infer 的后端程序进行分析。 The files contain serialized OCaml data structures. The `.cfg` file contains a control flow graph for each function or method implemented in the file. The file `.cg` contains the call graph of the functions defined or called from that file. Finally, the file `.tenv` contains all the types that are defined or used in the file.

The files `.cfg`, `.cg` and `.tenv` contain the intermediate representation of that file. This data is passed to the backend of Infer, which then performs the analysis. The files contain serialized OCaml data structures. The `.cfg` file contains a control flow graph for each function or method implemented in the file. The file `.cg` contains the call graph of the functions defined or called from that file. Finally, the file `.tenv` contains all the types that are defined or used in the file.



## Debug mode

With the debug option enabled `infer --debug -- <build command>`, Infer outputs debug information. When using `make` or `clang`, one needs an extra debug argument for the frontend:

```bash
infer --frontend_debug --debug -- make example.c
```

In each captured folder, we obtain the file `icfg.dot`, which is the graphical representation of the file `.cfg` and the file
`call_graph.dot`, that is the graphical representation of the call graph.


Moreover, we obtain an html page for each captured file inside `infer-out/captured`. This html file contains the source file. In each line of the file there are links to the nodes of the control flow graph that correspond to that line of code. So one can see what the translation looks like. Moreover, when you click on those links you can see details of the symbolic execution of that particular node. If the option `--no_test` is also passed to `infer`, then the page pointed to from the nodes contains the printout of the whole symbolic execution.

## Print the specs

It is also possible to print the specs created by Infer using the command `InferPrint`. You can print one particular spec that corresponds to one method, or you can print all the specs in the results directory. Let us look at an example:

```java
class Hello {
    int x;
    void setX(int newX) {
	    this.x = newX;
    }
}
```

We run Infer on this example with:

```bash
	infer -- javac Hello.java
```

Infer saves the spec for the method `setX` in `infer-out/specs` and we can print it with the command:

```bash
	InferPrint infer-out/specs/Hello.setX{98B5}:void.specs
```

The convention for method names in Java is `<class name>.<method name>`. This outputs the following:

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

which expresses the fact that `this` needs to be allocated at the beginning of the method, and that at the end of the method the field `x` is equal to `newX`.


Moreover, you can print all the specs in the results directory with the command:

```bash
InferPrint -results_dir infer-out
```


## Run internal tests

There are many tests in the Infer code base that check that Infer behaves correctly on small program examples. The tests use [Buck](http://buckbuild.com/), another Facebook's open source tool for building projects. We provide the script `inferTest` to run the tests, which requires buck to be in your PATH.

```bash
inferTest java    # Run the tests about Java analysis
inferTest clang   # Run the tests about C and Objective-C analysis
inferTest c       # Run the tests about C analysis
inferTest objc    # Run the tests about Objective-C analysis
```
