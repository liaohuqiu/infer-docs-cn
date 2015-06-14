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

通常对于一个对象，如果你什么都没说明，默认认为这个对象不会是空值。在可能的情况下，我们建议：

安全编程，注解空值。

如果可能为空值，即为类型参数加上 @Nullable 注解。

### 什么是注解

注解放在方法调用或者成员变量访问的接口中：

- 定义方法时的参数和返回值类型
- 成员变量申明

局部变量没有办法加注解，他们的可空性是推断出来的。

### Infer:Eradicate 如何调用？

通过 `-a eradicate` 选项，可以启用 Eradicate，如下：

```bash
infer -a eradicate -- javac Test.java
```

对于这样的代码，试图访问一个可空的值，却没有做空检查，检测器会检查并报告错误。

```java
class C {
  int getLength(@Nullable String s) {
    return s.length();
  }
}
```

但如果是以下这样，那么就没问题：

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
