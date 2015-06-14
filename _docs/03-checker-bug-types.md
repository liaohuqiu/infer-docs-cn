---
id: checkers-bug-types
title: 校验器报告的问题类型
layout: docs
permalink: /docs/checkers-bug-types.html
section: Bug Types Reference
section_order: 03
order: 02
---

## <a name="CHECKERS_IMMUTABLE_CAST"></a>不可变转化校验器

这个问题会在 Java 代码中出现。当函数返回类型是一个可变集合，但是实际返回的是不可变集合。

```java
  public List<String> getSomeList() {
    ImmutableList<String> l = foo(...);
    return l;
  }
```

如果对 `getSomeList` 的返回值进行操作，比如添加一些数据项，将会触发运行时错误。

至于修改方案，你可以修改返回类型，也可以把集合数据拷贝一份，变成可变集合。
