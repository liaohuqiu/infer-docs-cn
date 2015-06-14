---
id: eradicate
title: "Infer : Eradicate"
layout: docs
permalink: /docs/eradicate.html
section: User Guide
section_order: 01
order: 04
---


> "I call it my billion-dollar mistake. It was the invention of the null reference in 1965."
> 
> [Tony Hoare](http://en.wikipedia.org/wiki/Tony_Hoare)


### 什么是 Infer:Eradicate ？

Infer:Eradicate 是针对Java @Nullable 注解的一个检查器，是 Infer 静态分析工具套件中的一部分，目标是消除空指针异常。

<a href="https://developer.android.com/reference/android/support/annotation/Nullable.html">@Nullable</a> 注解指示一个参数，类成员，或者方法返回值可以是 null。

当这个注解修饰一个参数时，说明这个参数是允许为空的，方法体内部应该处理为空的情况。

当注解修饰一个参数时，说明方法的返回值是可以为空的。

从标注为 `@Nullable` 的程序开始，可空性将随着赋值和调用进行传播，分析器对这个流程敏感的传播过程进行分析。

分析之后，对那些未受保护的空值访问，前后不一致的@Nullable 注解或者该标记却没标记的方法或变量，加上错误标记。

Infer:Eradicate 也用来将之前未标记注解的代码添加注解。

### 什么是 @Nullable 约定？


If you say nothing, you're saying that the value cannot be null. This is the recommended option when possible:

Program safely, annotate nothing!

When this cannot be done, add a @Nullable annotation before the type to indicate that the value can be null.

### What is annotated?

Annotations are placed at the interface of method calls and field accesses:

- Parameters and return type of a method declaration.
- Field declarations.

Local variable declarations are not annotated: their nullability is inferred.

### Infer:Eradicate 如何调用？

Eradicate can be invoked by adding the option `-a eradicate` to the analysis command as in this example:

```bash
infer -a eradicate -- javac Test.java
```

The checker will report an error on the following program that accesses a nullable value without null check:

```java
class C {
  int getLength(@Nullable String s) {
    return s.length();
  }
}
```

But it will not report an error on this guarded dereference:

```java
class C {
  int getLength(@Nullable String s) {
    if (s != null) {
      return s.length();
    } else {
      return -1;
    }
  }
}
```

Eradicate 会输出这些[警告](/docs/eradicate-warnings.html).
