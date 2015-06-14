---
id: checkers
title: "Infer : Checkers"
layout: docs
permalink: /docs/checkers.html
section: User Guide
section_order: 01
order: 03
---

Infer 的分析器执行复杂的程序间（interprocedural，专注整体）静态分析。但当我们针对那些 linter 中常见的分析，不需要复杂的程序间的分析的时候，我们有一个称为 Infer:Checkers（Infer 校验器） 的框架。

Infer:Checkers 可以检测给定项目中每个方法的某个指定属性，虽然分析了整个项目，但是这种分析算是程序内的而不是程序间的。

通过选项 `-a checkers` 可以在分析时加入校验器(checkers)，如下：

```bash
infer -a checkers -- javac Test.java
```

目前，我们有[不可变转化校验器](docs/checkers-bug-types.html#CHECKERS_IMMUTABLE_CAST). 
