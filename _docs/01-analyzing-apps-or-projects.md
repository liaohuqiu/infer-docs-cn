---
id: analyzing-apps-or-projects
title: 分析 APP 或者其他项目
layout: docs
permalink: /docs/analyzing-apps-or-projects.html
section: User Guide
section_order: 01
order: 02
---

使用 Infer 分析文件时，你可以使用 `javac` 或者 `clang` 编译器，当然你也可以使用 `gcc`。但在 Infer 内部会使用 `clang` 去编译你的代码。所以如果你的代码无法用 `clang` 编译的话，可能无法使用 Infer。

除此之外，你还可以和其他许多编译系统一起使用 Infer。注意一点，你可以通过并行编译命令来加快 Infer 的运行检测，比如：`infer -- make -j8`。

如果你想分析整个项目的话，分析之前，记得清理项目。这样编译器才会重新编译所有文件，Infer 才会分析这些编译的文件（具体看 [这个章节](docs/infer-workflow.html)）。

以下是 Infer 目前支持的编译系统。对于某个特定的系统，你可以通过 `infer --help -- <build system>` 了解更多具体的信息。比如： `infer --help -- gradle`。

### Gradle

```bash
infer -- gradle <gradle task, e.g. "build">
infer -- ./gradlew <gradle task, e.g. "build">
```

### Buck

```bash
infer -- buck <buck target>
```

### Maven
```bash
infer -- mvn <maven target>
```

### Xcodebuild

Infer 可是分析使用 `xcodebuild` 构建的应用，但是只分析 `.m` 和 `.c` 文件，其他的文件，比如：`.cpp`，`.cc`，`.mm` 文件会被忽略。

比如一个 iOS 应用：

```bash
infer -- xcodebuild -target <target name> -configuration <build configuration> -sdk iphonesimulator
```

### Make

Infer 可以分析使用 `make` 构建的项目，项目中的 C++ 文件会被忽略。

```bash
infer -- make <make target>
```
